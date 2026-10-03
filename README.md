# Vortex Go Style Guide

Standard dated **October 3, 2026**.

---

This document establishes a unified standard for developing and formatting Go code across the **Vortex** ecosystem.
Compliance with these rules is mandatory for all services and components of the organization.

This document is based on the recommendations in [Effective Go](https://golang.org/doc/effective_go) and [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments).

**Priority hierarchy:**
1. The rules and architectural requirements explicitly described in this Standard (**Vortex Go Style Guide**) take absolute precedence.
2. For topics not covered by this document, follow *Effective Go* and *Go Code Review Comments* strictly.

## Table of Contents

- [Vortex Go Style Guide](#vortex-go-style-guide)
  - [Table of Contents](#table-of-contents)
  - [1. General Rules and Function Signatures](#1-general-rules-and-function-signatures)
    - [1.1. Prohibition on the `init()` Function](#11-prohibition-on-the-init-function)
    - [1.2. Passing `context.Context`](#12-passing-contextcontext)
    - [1.3. Position of `error` in Return Values](#13-position-of-error-in-return-values)
  - [2. File Structure](#2-file-structure)
    - [2.1. Order of Declarations in a File](#21-order-of-declarations-in-a-file)
    - [2.2. Constructors and Parameter Structures (Params)](#22-constructors-and-parameter-structures-params)
  - [3. Naming Conventions](#3-naming-conventions)
    - [3.1. Errors](#31-errors)
    - [3.2. CRUD Functions](#32-crud-functions)
  - [4. Comments](#4-comments)
    - [4.1. General Comment Requirements](#41-general-comment-requirements)
    - [4.2. Hyperlinks](#42-hyperlinks)
    - [4.3. Function and Method Comment Structure](#43-function-and-method-comment-structure)
    - [4.4. Exceptions and Relaxations](#44-exceptions-and-relaxations)
      - [No Arguments or Results](#no-arguments-or-results)
      - [Short Functions](#short-functions)
      - [Obvious Arguments](#obvious-arguments)
      - [Logical Blocks and Wrapping Long Lines](#logical-blocks-and-wrapping-long-lines)
    - [4.5. Documenting Returned Errors](#45-documenting-returned-errors)
    - [4.6. Documenting Error Variables](#46-documenting-error-variables)
  - [5. Testing](#5-testing)
    - [5.1. Visibility Restrictions](#51-visibility-restrictions)
    - [5.2. Field Order in Table-Driven Tests](#52-field-order-in-table-driven-tests)
  - [6. Error Handling](#6-error-handling)
    - [6.1. Wrapping and Logging Errors](#61-wrapping-and-logging-errors)
    - [6.2. Aggregating Errors with `errors.Join()`](#62-aggregating-errors-with-errorsjoin)
    - [6.3. Using `errors.New()` for Static Errors](#63-using-errorsnew-for-static-errors)
    - [6.4. Collecting Errors in `Validate()`](#64-collecting-errors-in-validate)
    - [6.5. Error Messages](#65-error-messages)
      - [Required Resource Errors](#required-resource-errors)
      - [Resource Not Found Errors](#resource-not-found-errors)

---

## 1. General Rules and Function Signatures

### 1.1. Prohibition on the `init()` Function

The use of the `init()` function is strictly prohibited. Variables, dependencies, and configuration must be initialized explicitly through constructors or initializer calls in the `main` / `app` packages.

> Reason: `init()` hides initialization order, complicates testing, and creates implicit side effects when a package is imported.

### 1.2. Passing `context.Context`

If a function or method accepts `context.Context`, it **must always** be the first argument.

```go
// Good
func (s *Service) FetchUser(ctx context.Context, userID string) (*User, error)

// Bad
func (s *Service) FetchUser(userID string, ctx context.Context) (*User, error)
```

### 1.3. Position of `error` in Return Values

If a function returns an error value (`error`), it must always be the last value in the return list.

```go
// Good
func Load(path string) (*Config, error)

// Bad
func Load(path string) (error, *Config)
```

## 2. File Structure

### 2.1. Order of Declarations in a File

Every `.go` file must follow this strict order of code declarations:

1. File-level constants (`const`) and variables (`var`).
2. Custom types and interfaces (`type`).
3. Public methods on types (`func (t *Type) PublicMethod()`).
4. Private methods on types (`func (t *Type) privateMethod()`).
5. Public functions in the file (`func PublicFunction()`).
6. Private functions in the file (`func privateFunction()`).

```go
package example

// 1. File-level const and var declarations
const defaultTimeout = 5

var packageVersion = "1.0"

// 2. Types
type Service struct {
	timeout time.Duration
}

// 3. Public methods on types
func (s *Service) Timeout() time.Duration { return s.timeout }

// 4. Private methods on types
func (s *Service) setTimeout(t time.Duration) {
	s.timeout = t
}

// 5. Public functions in the file
func PrintPackageVersion() {
	fmt.Printf("version: %s", packageVersion)
}

// 6. Private functions in the file
func formatVersion() string {
	return "v" + packageVersion
}
```

### 2.2. Constructors and Parameter Structures (Params)

Types used as parameter containers for constructing a struct (the *Params pattern*) must be declared at the end of the custom types block.

The type's constructor must immediately follow the `Params` struct and its methods:

```go
type Service struct {
	logger Logger
	db     Database
}

// Params is declared at the end of the custom types block.
type ServiceParams struct {
	Logger Logger
	DB     Database
}

func (p *ServiceParams) Validate() error {
	if p.Logger == nil {
		return errors.New("logger is required")
	}
	return nil
}

// The constructor appears immediately after ServiceParams.
func NewService(params *ServiceParams) (*Service, error) {
	if err := params.Validate(); err != nil {
		return nil, fmt.Errorf("invalid service params: %w", err)
	}

	return &Service{
		logger: params.Logger,
		db:     params.DB,
	}, nil
}
```

## 3. Naming Conventions

### 3.1. Errors

All error variables, both exported and private, must begin with the `Err` (or `err`) prefix.

```go
// Good
var (
	ErrEmptyURL     = errors.New("URL is required")
	ErrUserNotFound = errors.New("user is required")
)

// Bad
var (
	BadURLErr = errors.New("URL cannot be empty")
)
```

### 3.2. CRUD Functions

Functions and methods that interact with databases and storage systems must follow an explicit prefix convention.

> Unlike the general *Effective Go* recommendations, which discourage the **Get** prefix for ordinary getter methods, using **Get** for **CRUD operations** is permitted and recommended to improve readability in the data layer.

**Required CRUD prefixes:**

- `Get` — retrieve or read existing records from a database (**GetByID**, **GetByName**).
- `Insert` — insert or create new records in a database (**InsertUser**, **InsertSession**).
- `Update` — modify or update existing data (**UpdateStatus**, **UpdateProfile**).
- `Delete` — physically or logically delete records (**DeleteByID**, **DeleteSoft**).

```go
// Good: CRUD operations in the data layer have clear prefixes.
func (r *Repository) GetUserByID(ctx context.Context, id string) (*User, error)
func (r *Repository) InsertUser(ctx context.Context, user *User) error
func (r *Repository) UpdateStatus(ctx context.Context, id string, status Status) error
func (r *Repository) DeleteByID(ctx context.Context, id string) error

// Bad: Non-standard verbs are used for database operations.
func (r *Repository) SaveUser(ctx context.Context, user *User) error   // Use Insert.
func (r *Repository) RemoveUser(ctx context.Context, id string) error  // Use Delete.
func (r *Repository) FetchUser(ctx context.Context, id string) error   // Use Get.
```

## 4. Comments

Code documentation must follow strict **GoDoc** standards. The primary purpose of comments is to explain **WHY** an architectural or business decision was made, not to restate **HOW** the code works.

### 4.1. General Comment Requirements

1. **Language:** All comments must be written in English.
2. **100% coverage:** Comments are **required** for every entity in a file, exported and private: **functions**, **methods**, **structs**, **interfaces**, **interface methods**, **custom types**, **constants**, and **variables**.
3. **Formatting:** A documentation comment for an exported entity (and preferably a private one) must begin with its name.
4. **Punctuation:** Every sentence must end with a period, whether it is an entity description or an inline comment.

### 4.2. Hyperlinks

To create hyperlinks, enclose types in square brackets: **[Type]** or **[pkg.Type]**.
**Do not** enclose the entity being documented in a hyperlink:

```go
// Good comment:

// ServiceParams encapsulates the necessary dependencies for creating a [Service] instance.
type ServiceParams struct{}

// Bad comment:

// [ServiceParams] encapsulates the necessary dependencies for creating a [Service] instance.
type ServiceParams struct{}
```

### 4.3. Function and Method Comment Structure

Function documentation must follow this strict logical structure:

- **Line 1:** A brief description of the function or method's purpose, beginning with its name.
- **Line 2:** A description of the arguments and why they are required, written as one connected sentence, with arguments separated by commas and no colons.
- **Line 3:** A description of the result, beginning with *"It returns..."* or an equivalent phrase.
- **Additional notes:** If needed, insert a blank `//` line after **Line 3** and write each note on a new line.

```go
// ConvertMedia transcodes the incoming media stream into the target format.
// Takes ctx for boundary cancellation, payload containing raw video bytes, and opts for configuration.
// It returns the converted [MediaResult] or an error if processing fails.
//
// This operation is resource-intensive and triggers CPU-bound workers in Vortex Engine.
func (s *Service) ConvertMedia(ctx context.Context, payload []byte, opts *ConvertOptions) (*MediaResult, error)
```

### 4.4. Exceptions and Relaxations

#### No Arguments or Results

If a function accepts no arguments or returns nothing, omit the corresponding lines from the comment structure.

#### Short Functions

For simple functions and methods whose implementation is no more than five lines of code, a single-line GoDoc comment is permitted. Details about arguments and return values may be combined or omitted entirely.

```go
// Good:

// threadsArgs returns FFmpeg CLI flags to configure multi-threaded processing.
func (c *ffmpegConverter) threadsArgs() []string {
	return []string{"-threads", strconv.Itoa(int(c.maxThreads)), "-filter_threads", strconv.Itoa(int(c.maxThreads))}
}

// Bad:

// threadsArgs Creates a slice of FFmpeg CLI arguments to control the number of threads used.
// The method takes no arguments.
// Returns a slice containing the FFmpeg arguments.
func (c *ffmpegConverter) threadsArgs() []string {
	return []string{"-threads", strconv.Itoa(int(c.maxThreads)), "-filter_threads", strconv.Itoa(int(c.maxThreads))}
}
```

#### Obvious Arguments

A separate line describing the arguments (**Line 2**) is not required when the purpose and necessity of the parameters are unambiguous from their names.

```go
// Good:

// Load reads and parses the JSON configuration at path.
// It returns the pointer to the parsed [Config] instance and an error if path is empty or reading or parsing fails.
func Load(path string) (*Config, error) {...}

// Bad:

// Load reads and parses the JSON configuration at path.
// Takes path as the path to the configuration file.
// It returns the pointer to the parsed [Config] instance and an error if path is empty or reading or parsing fails.
func Load(path string) (*Config, error) {...}
```

#### Logical Blocks and Wrapping Long Lines

The labels (**Line 1**, **Line 2**, **Line 3**) refer to logical blocks in a **GoDoc comment**, not to a requirement that each block occupy exactly one physical line of code.

If the argument description (**Block 2**) or return-value description (**Block 3**) exceeds a comfortable line length (**120–150 characters**), it is permitted and recommended to wrap it onto the next line while preserving the context.

### 4.5. Documenting Returned Errors

If a function or method returns specific errors that are not created inside that function, they must be documented in GoDoc using **hyperlinks** on the **third line** of the comment.

> This ensures that callers know which error types and variables they can handle with `errors.Is()` or `errors.As()`.

```go
var ErrMediaNotFound = errors.New("media not found")

var ErrAccessDenied = errors.New("access denied")

// FetchMedia retrieves the processed media content by its unique identifier.
// Takes ctx for timeout management and id representing the target media resource.
// It returns an [ErrMediaNotFound] if the record is missing, [ErrAccessDenied] if permissions are insufficient, any errors on validation failure, or the target [Media] if there are no errors.
func (s *Service) FetchMedia(ctx context.Context, id string) (*Media, error) {}
```

### 4.6. Documenting Error Variables

Comments for error variables, both public and private, must strictly follow the template **"... is triggered when ..."** and clearly state the conditions under which the error occurs.

```go
// ErrEmptyURL is triggered when there is an empty URL string in the Source payload structure.
var ErrEmptyURL = errors.New("URL is required")

// errTimeoutExceeded is triggered when the Vortex Engine fails to respond within the allocated deadline.
var errTimeoutExceeded = errors.New("timeout exceeded")
```

## 5. Testing

### 5.1. Visibility Restrictions

Test files (`*_test.go`) may declare **only** unexported types, methods, and functions (those beginning with a lowercase letter).

```go
// Good
type mockUserRepo struct{}
func (m *mockUserRepo) getUser() {}
func createTestUser() *User {}

// Bad: Exported types are prohibited in test files.
type MockUserRepo struct{}
```

### 5.2. Field Order in Table-Driven Tests

When using table-driven tests, the case struct must follow this strict field order:

1. `name string` must always be the first field in the struct.
2. `wantErr bool` must always be the second field **if the tested method returns an error**.
3. `Context (ctx)` must be declared at the very end of the test case struct **if the tested method requires a context**.

## 6. Error Handling

### 6.1. Wrapping and Logging Errors

- When passing an error up the call stack, always wrap it with `fmt.Errorf()` using the `%w` verb.
- Use `%v` only to format error text for logging or console output.

```go
err := service.CreateUser(id)

// Pass the error up the call stack.
if err != nil {
	return fmt.Errorf("failed to create user: %w", err)
}

// Log the error.
log.Printf("failed to create user: %v", err)
```

### 6.2. Aggregating Errors with `errors.Join()`

If a function accumulates errors in a `[]error` slice to combine them later, return the result directly with `errors.Join(errs...)`. **There is no need to check `if len(errs) > 0`.**

```go
func (s *Service) Validate() error {
	var errs []error

	if s.logger == nil {
		errs = append(errs, errors.New("logger is required"))
	}

	if s.db == nil {
		errs = append(errs, errors.New("db is required"))
	}

	return errors.Join(errs...)
}
```

### 6.3. Using `errors.New()` for Static Errors

Whenever an error is created without embedding dynamic information (formatted variables), it must be created with `errors.New()`. **Using `fmt.Errorf()` for static text is prohibited.**

```go
// Good
var ErrInvalidToken = errors.New("token is invalid")

// Bad
var ErrInvalidPassword = fmt.Errorf("password is invalid")
```

### 6.4. Collecting Errors in `Validate()`

If a struct's `Validate()` method checks multiple conditions and may find more than one error, it must collect all errors found in a `[]error` slice and return them through `errors.Join(errs...)`.

Returning immediately on the first error (with an early `return err`) is prohibited because callers must receive a complete picture of the object's invalid state in a single call.

```go
// Good #1
func (p *ServiceParams) Validate() error {
	var errs []error

	if p.Logger == nil {
		errs = append(errs, errors.New("logger is required"))
	}

	if p.DB == nil {
		errs = append(errs, errors.New("database is required"))
	}

	return errors.Join(errs...)
}

// Good #2
// Only one field is checked here, so a []error slice would be unnecessary.
func (s *Service) Validate() error {
	if s.db == nil {
		return errors.New("database is required")
	}
	return nil
}

// Bad
func (p *ServiceParams) Validate() error {
	if p.Logger == nil {
		return errors.New("logger is required")
	}

	if p.DB == nil {
	   return errors.New("database is required")
	}

	return nil
}
```

### 6.5. Error Messages

#### Required Resource Errors

Validation error text indicating that a required field or resource is missing must end with **"is required"**.

```go
// Good
var ErrEmptyURL = errors.New("URL is required")

// Bad
var ErrEmptyUser = errors.New("user cannot be nil")
```

#### Resource Not Found Errors

Validation error text indicating that a resource was not found must end with **"not found"**.

> An empty argument passed to a **function or method** is not considered a resource that was not found.

For example, a required document missing from a database is considered a resource that was not found.

```go
// Good
var ErrUserNotFound = errors.New("user not found")

// Bad
var ErrResourceNotFound = errors.New("resource is required")
```