# Go > How to Publish Open Source Packages

Publishing a reusable Go package allows you to share code across multiple projects, contribute to the open source community, and establish standardized solutions to common problems. Unlike internal packages that live within a single project, published packages must be carefully designed for stability, clarity, and backward compatibility since external users depend on them.

This tutorial covers the complete lifecycle of publishing and maintaining Go packages: from initial design through versioning strategies to local development workflows.

## Prerequisites

Before publishing a package, you should have:

- A GitHub account (or another Git hosting service)
- Basic understanding of Go modules and the `go.mod` file
- Familiarity with Git commands (commit, tag, push)
- Knowledge of semantic versioning principles

## What Makes a Good Package

Not all code should be extracted into a package. Consider these criteria:

**When to publish:**
- The code solves a general problem, not business-specific logic
- Multiple projects would benefit from reusing this code
- The API is stable enough that breaking changes will be rare
- You're willing to maintain it (bug fixes, updates, documentation)

**Design principles:**
- **Single responsibility**: A package should do one thing well
- **Minimal dependencies**: Each dependency adds maintenance burden to users
- **Clear naming**: Package names should be short, lowercase, and descriptive (avoid `util`, `helper`, `common`)
- **Exported API**: Only export types and functions that users need (use lowercase for internal implementation)

## Creating Your First Package

The critical requirement for publishing a Go package is that the module name must match the GitHub repository path. This allows Go's tooling to automatically fetch your package.

Let's create a package that provides string manipulation utilities:

```bash
# Create a directory for your package
mkdir stringutil
cd stringutil

# Initialize the module with GitHub path
go mod init github.com/yourusername/stringutil
```

The module path format is `github.com/<username>/<repository-name>`. This exact path will be used by others to import your package.

Create the main package file:

```go
// stringutil.go
package stringutil

import "strings"

// Reverse returns the reversed version of the input string.
// Example: Reverse("hello") returns "olleh"
func Reverse(s string) string {
	runes := []rune(s)
	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
		runes[i], runes[j] = runes[j], runes[i]
	}
	return string(runes)
}

// Title converts a string to title case.
// Example: Title("hello world") returns "Hello World"
func Title(s string) string {
	return strings.Title(s)
}
```

**Key observations:**
- Function names start with uppercase letters (`Reverse`, `Title`) to make them exported
- Each exported function has a documentation comment starting with the function name
- Package name matches the repository name for simplicity

## Documentation Best Practices

Go has built-in documentation through `godoc`. Every exported type, function, constant, and variable should have a comment explaining its purpose.

**Documentation rules:**
- Comments must start with the name of the element being documented
- Use complete sentences with proper punctuation
- Provide usage examples in comments when behavior isn't obvious
- Package-level documentation goes in a comment block above the package declaration

Add package-level documentation:

```go
// Package stringutil provides utility functions for string manipulation.
// It includes functions for reversing strings, case conversion, and formatting.
package stringutil
```

**For complex packages**, create a `doc.go` file:

```go
// doc.go

// Package stringutil provides utility functions for string manipulation.
//
// This package aims to extend the standard library's strings package with
// commonly needed operations that aren't included by default.
//
// Basic usage:
//
//	reversed := stringutil.Reverse("hello")
//	// reversed is now "olleh"
//
//	titled := stringutil.Title("hello world")
//	// titled is now "Hello World"
//
// All functions handle Unicode correctly and work with multi-byte characters.
package stringutil
```

Users can view your documentation locally using `go doc`:

```bash
go doc github.com/yourusername/stringutil
go doc github.com/yourusername/stringutil.Reverse
```

Once published, it will also appear on pkg.go.dev automatically.

## Testing Your Package

Published packages should include comprehensive tests. Users trust packages with good test coverage more than untested code.

Create a test file:

```go
// stringutil_test.go
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	tests := []struct {
		name     string
		input    string
		expected string
	}{
		{"empty string", "", ""},
		{"single char", "a", "a"},
		{"simple word", "hello", "olleh"},
		{"unicode", "Hello, 世界", "界世 ,olleH"},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			result := Reverse(tt.input)
			if result != tt.expected {
				t.Errorf("Reverse(%q) = %q, want %q", tt.input, result, tt.expected)
			}
		})
	}
}

func TestTitle(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
		{"hello", "Hello"},
		{"hello world", "Hello World"},
		{"", ""},
	}

	for _, tt := range tests {
		result := Title(tt.input)
		if result != tt.expected {
			t.Errorf("Title(%q) = %q, want %q", tt.input, result, tt.expected)
		}
	}
}
```

Run tests with:

```bash
go test -v
```

**Testing best practices:**
- Use table-driven tests to cover multiple cases efficiently
- Include edge cases (empty strings, Unicode, special characters)
- Aim for high coverage of exported functions
- Add benchmarks for performance-critical code using `Benchmark` functions

## Publishing to GitHub

Once your package is ready, publish it to GitHub:

```bash
# Initialize git repository
git init

# Create .gitignore
echo "# Binaries
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test coverage
*.out" > .gitignore

# Add and commit files
git add .
git commit -m "Initial commit: Add string utilities package"

# Create repository on GitHub, then add remote
git remote add origin https://github.com/yourusername/stringutil.git

# Push to GitHub
git branch -M main
git push -u origin main
```

**Create the first version tag:**

```bash
git tag v1.0.0
git push --tags
```

The tag must start with `v` followed by semantic version number. This tag tells Go's module system that version `1.0.0` is available.

## Using Your Published Package

Other projects can now use your package:

```bash
# In another project
go get github.com/yourusername/stringutil@v1.0.0
```

The `@v1.0.0` part is optional but recommended for reproducible builds. Without it, Go fetches the latest version.

Import and use it:

```go
package main

import (
	"fmt"
	"github.com/yourusername/stringutil"
)

func main() {
	fmt.Println(stringutil.Reverse("Hello, World!"))
	// Output: !dlroW ,olleH
}
```

## Understanding Semantic Versioning

Semantic versioning (semver) is a versioning scheme that communicates the nature of changes through version numbers. The format is `MAJOR.MINOR.PATCH`:

- **MAJOR** (v1.0.0 → v2.0.0): Incompatible API changes that break existing code
- **MINOR** (v1.0.0 → v1.1.0): New functionality added in a backward-compatible manner
- **PATCH** (v1.0.0 → v1.0.1): Backward-compatible bug fixes

**Examples:**
- Fixing a bug in `Reverse()`: v1.0.0 → v1.0.1 (patch)
- Adding a new `ToSnakeCase()` function: v1.0.0 → v1.1.0 (minor)
- Changing `Reverse()` signature from `(string) string` to `(string) (string, error)`: v1.0.0 → v2.0.0 (major)

**Why this matters:**
Go modules use semantic versioning to determine compatibility. When you run `go get -u`, Go updates to the latest minor and patch versions but never crosses major version boundaries automatically. This prevents unexpected breaking changes.

## Making Updates

Let's add a new function to our package:

```go
// stringutil.go
package stringutil

// ... existing code ...

// ToSnakeCase converts a string to snake_case.
// Example: ToSnakeCase("HelloWorld") returns "hello_world"
func ToSnakeCase(s string) string {
	var result []rune
	for i, r := range s {
		if i > 0 && unicode.IsUpper(r) {
			result = append(result, '_')
		}
		result = append(result, unicode.ToLower(r))
	}
	return string(result)
}
```

Since this adds functionality without breaking existing code, it's a minor version bump:

```bash
git add .
git commit -m "Add ToSnakeCase function"
git tag v1.1.0
git push
git push --tags
```

Users can update to the new version:

```bash
go get github.com/yourusername/stringutil@v1.1.0
# Or get the latest version
go get -u github.com/yourusername/stringutil
```

## Managing Major Versions

Major version changes (v2, v3, etc.) require special handling in Go because they represent incompatible API changes. Go treats different major versions as distinct modules, allowing users to depend on multiple major versions simultaneously.

**When to create a major version:**
- Removing or renaming exported functions/types
- Changing function signatures
- Altering behavior in ways that could break existing code
- Removing support for deprecated features

**Example scenario:** Let's say we want to make `Reverse()` return an error for invalid UTF-8:

```go
// Breaking change: function signature changed
func Reverse(s string) (string, error) {
	if !utf8.ValidString(s) {
		return "", errors.New("invalid UTF-8 string")
	}
	// ... rest of implementation
}
```

This breaks existing code that expects `reversed := stringutil.Reverse("hello")`.

### Approach 1: Subdirectories

Create a `v2` directory and move all code into it:

```text
stringutil/
├── README.md
├── go.mod          # v1 module
├── stringutil.go   # v1 code
├── stringutil_test.go
└── v2/
    ├── go.mod      # v2 module
    ├── stringutil.go   # v2 code
    └── stringutil_test.go
```

Update `v2/go.mod`:

```text
module github.com/yourusername/stringutil/v2

go 1.21
```

Commit and tag:

```bash
git add .
git commit -m "Release v2.0.0 with error handling"
git tag v2.0.0
git push
git push --tags
```

**Using v2:**

```bash
go get github.com/yourusername/stringutil/v2@v2.0.0
```

```go
import "github.com/yourusername/stringutil/v2"

reversed, err := stringutil.Reverse("hello")
if err != nil {
	log.Fatal(err)
}
```

**Using v1 and v2 simultaneously:**

```go
import (
	stringutilv1 "github.com/yourusername/stringutil"
	stringutilv2 "github.com/yourusername/stringutil/v2"
)

// Use v1 for simple cases
old := stringutilv1.Reverse("hello")

// Use v2 where error handling is needed
new, err := stringutilv2.Reverse(userInput)
```

### Approach 2: Branches

Instead of subdirectories, use Git branches:

```bash
# Create v3 branch
git checkout -b v3

# Update go.mod
echo 'module github.com/yourusername/stringutil/v3

go 1.21' > go.mod

# Make breaking changes to code
# ...

git add .
git commit -m "Release v3.0.0 with new API"
git tag v3.0.0
git push origin v3
git push --tags
```

**Trade-offs:**
- **Subdirectories**: All versions visible in main branch, easier to maintain multiple versions
- **Branches**: Cleaner repository structure but harder to backport fixes across versions

**Most projects use subdirectories** because it's easier to maintain and more discoverable.

## Package Organization with Subpackages

As packages grow, you may want to split functionality into subpackages:

```text
stringutil/
├── go.mod
├── stringutil.go      # Core functions
├── case/
│   └── case.go        # Case conversion functions
└── validation/
    └── validation.go  # String validation functions
```

Each subdirectory is a separate package:

```go
// case/case.go
package case

func ToSnakeCase(s string) string {
	// implementation
}
```

Users import subpackages explicitly:

```go
import (
	"github.com/yourusername/stringutil"
	"github.com/yourusername/stringutil/case"
)

reversed := stringutil.Reverse("hello")
snake := case.ToSnakeCase("HelloWorld")
```

**Updating after adding subpackages:**

```bash
git add .
git commit -m "Reorganize into subpackages"
git tag v1.2.0  # Minor version bump (backward compatible)
git push
git push --tags
```

Users update to get the new structure:

```bash
go get github.com/yourusername/stringutil@v1.2.0
```

## Best Practices and Considerations

### Security

- **Never commit secrets**: Avoid hardcoded API keys, passwords, or tokens
- **Validate inputs**: Especially for functions that interact with filesystems or networks
- **Document security considerations**: If your package handles sensitive data, document proper usage
- **Dependency auditing**: Regularly check dependencies for vulnerabilities using `go list -m all | nancy sleuth` or similar tools

### Performance

- **Profile before optimizing**: Use `pprof` to identify actual bottlenecks
- **Include benchmarks**: Add benchmark tests for performance-critical functions
- **Document complexity**: State time/space complexity for algorithms in documentation
- **Consider memory allocations**: Minimize allocations in hot paths

### Maintenance

- **Semantic versioning discipline**: Always follow semver rules strictly
- **Changelog**: Maintain a CHANGELOG.md documenting changes between versions
- **Deprecation grace period**: When removing features, deprecate them in one version and remove in the next major version
- **GitHub releases**: Create GitHub releases with release notes for each tag
- **Respond to issues**: Monitor and respond to user-reported issues

### API Design

- **Start small**: It's easier to add features than remove them
- **Accept interfaces, return structs**: Makes your package more flexible
- **Context awareness**: For I/O operations, accept `context.Context` as the first parameter
- **Error messages**: Provide clear, actionable error messages
- **Zero values**: Design types so their zero value is useful when possible

## Local Development with Go Workspaces

When developing a package locally before publishing, or when working on a package and its consumer simultaneously, Go workspaces provide a clean solution.

**Scenario:** You're developing `stringutil` and want to test it in an application before publishing.

Create a workspace:

```bash
# Create workspace directory
mkdir my-workspace
cd my-workspace

# Initialize workspace
go work init

# Create the library
mkdir stringutil
cd stringutil
go mod init github.com/yourusername/stringutil
# ... create package files ...

# Create the application
cd ..
mkdir myapp
cd myapp
go mod init myapp
# ... create main.go ...

# Add both to workspace
cd ..
go work use ./stringutil
go work use ./myapp
```

The `go.work` file looks like:

```text
go 1.21

use (
	./myapp
	./stringutil
)
```

Now `myapp` can import the local `stringutil`:

```go
// myapp/main.go
package main

import "github.com/yourusername/stringutil"

func main() {
	println(stringutil.Reverse("test"))
}
```

Run the app:

```bash
cd myapp
go run main.go
```

Go automatically uses the local version of `stringutil` from the workspace instead of trying to download it.

**Workspace benefits:**
- No need to publish changes to test them
- Work on multiple related modules simultaneously
- Changes in one module immediately available in others
- Clean separation between modules (each has its own `go.mod`)

**Important:** The `go.work` file is for local development only. Add it to `.gitignore` and never commit it. Users of your package shouldn't see or need your workspace configuration.

### Alternative: Replace Directive

For simpler cases or CI/CD environments, use the `replace` directive in `go.mod`:

```text
// myapp/go.mod
module myapp

go 1.21

require github.com/yourusername/stringutil v1.0.0

replace github.com/yourusername/stringutil => ../stringutil
```

The `replace` directive tells Go to use the local directory instead of fetching from the remote repository.

**When to use replace:**
- Testing unpublished changes
- Temporarily using a fork
- CI/CD environments where workspaces aren't suitable

**Remove the replace directive before committing** unless you have a specific reason to keep it (like permanently using a fork).

## Summary

Publishing a Go package involves:

1. **Design**: Create focused packages with clear responsibilities
2. **Documentation**: Write godoc comments for all exported elements
3. **Testing**: Provide comprehensive tests for reliability
4. **Versioning**: Follow semantic versioning strictly
5. **Publishing**: Tag releases and push to GitHub
6. **Maintenance**: Respond to issues and maintain backward compatibility

The Go module system makes package distribution straightforward, but good package design requires thoughtful API decisions and commitment to stability. Start with a clear problem to solve, document thoroughly, test comprehensively, and version responsibly.
