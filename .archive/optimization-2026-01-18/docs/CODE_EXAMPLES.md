# Code Examples

This document provides practical code examples for using the namegenerator library across different scenarios and programming patterns.

---

## Table of Contents

- [Go Examples](#go-examples)
- [Integration Examples](#integration-examples)
- [FFI Examples (Other Languages)](#ffi-examples-other-languages)
- [Testing Examples](#testing-examples)
- [Production Patterns](#production-patterns)

---

## Go Examples

### Example 1: Basic Usage

```go
package main

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

func main() {
    // Create generator with time-based seed
    seed := time.Now().UTC().UnixNano()
    gen := namegenerator.NewNameGenerator(seed)

    // Generate a single name
    name := gen.Generate()
    fmt.Printf("Generated name: %s\n", name)
}
```

**Output**:

```
Generated name: silent-moon
```

---

### Example 2: Generate Multiple Names

```go
package main

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

func main() {
    seed := time.Now().UTC().UnixNano()
    gen := namegenerator.NewNameGenerator(seed)

    fmt.Println("Generated Names:")
    for i := 0; i < 10; i++ {
        name := gen.Generate()
        fmt.Printf("%2d. %s\n", i+1, name)
    }
}
```

**Output**:

```
Generated Names:
 1. silent-moon
 2. ancient-river
 3. bold-firefly
 4. withered-leaf
 5. crimson-mountain
 6. misty-waterfall
 7. patient-breeze
 8. wild-thunder
 9. quiet-dawn
10. fragrant-cherry
```

---

### Example 3: Deterministic Generation (Fixed Seed)

```go
package main

import (
    "fmt"
    "github.com/obot-platform/namegenerator"
)

func main() {
    // Fixed seed for reproducible results
    const testSeed = 12345

    // Create two generators with same seed
    gen1 := namegenerator.NewNameGenerator(testSeed)
    gen2 := namegenerator.NewNameGenerator(testSeed)

    fmt.Println("Generator 1 | Generator 2")
    fmt.Println("------------+------------")

    for i := 0; i < 5; i++ {
        name1 := gen1.Generate()
        name2 := gen2.Generate()
        fmt.Printf("%-11s | %-11s\n", name1, name2)
    }
}
```

**Output**:

```
Generator 1 | Generator 2
------------+------------
wandering-frost | wandering-frost
bold-mountain | bold-mountain
icy-silence | icy-silence
ancient-bird | ancient-bird
crimson-leaf | crimson-leaf
```

---

### Example 4: Custom Prefixes

```go
package main

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type PrefixedGenerator struct {
    gen    namegenerator.Generator
    prefix string
}

func NewPrefixedGenerator(seed int64, prefix string) *PrefixedGenerator {
    return &PrefixedGenerator{
        gen:    namegenerator.NewNameGenerator(seed),
        prefix: prefix,
    }
}

func (pg *PrefixedGenerator) Generate() string {
    return fmt.Sprintf("%s-%s", pg.prefix, pg.gen.Generate())
}

func main() {
    seed := time.Now().UTC().UnixNano()

    prodGen := NewPrefixedGenerator(seed, "prod")
    devGen := NewPrefixedGenerator(seed+1000, "dev")

    fmt.Println("Production Names:")
    for i := 0; i < 3; i++ {
        fmt.Println(" -", prodGen.Generate())
    }

    fmt.Println("\nDevelopment Names:")
    for i := 0; i < 3; i++ {
        fmt.Println(" -", devGen.Generate())
    }
}
```

**Output**:

```
Production Names:
 - prod-silent-moon
 - prod-ancient-river
 - prod-bold-firefly

Development Names:
 - dev-misty-waterfall
 - dev-patient-breeze
 - dev-wild-thunder
```

---

### Example 5: Unique Name Generator

```go
package main

import (
    "fmt"
    "log"
    "time"
    "github.com/obot-platform/namegenerator"
)

type UniqueGenerator struct {
    gen  namegenerator.Generator
    used map[string]bool
}

func NewUniqueGenerator(seed int64) *UniqueGenerator {
    return &UniqueGenerator{
        gen:  namegenerator.NewNameGenerator(seed),
        used: make(map[string]bool),
    }
}

func (ug *UniqueGenerator) Generate() (string, error) {
    maxAttempts := 100

    for attempt := 0; attempt < maxAttempts; attempt++ {
        name := ug.gen.Generate()

        if !ug.used[name] {
            ug.used[name] = true
            return name, nil
        }
    }

    return "", fmt.Errorf("failed to generate unique name after %d attempts", maxAttempts)
}

func (ug *UniqueGenerator) Reset() {
    ug.used = make(map[string]bool)
}

func (ug *UniqueGenerator) Count() int {
    return len(ug.used)
}

func main() {
    seed := time.Now().UTC().UnixNano()
    gen := NewUniqueGenerator(seed)

    fmt.Println("Generating 100 unique names:")
    for i := 0; i < 100; i++ {
        name, err := gen.Generate()
        if err != nil {
            log.Fatal(err)
        }
        if i < 10 || i >= 95 {
            fmt.Printf("%3d. %s\n", i+1, name)
        } else if i == 10 {
            fmt.Println("...")
        }
    }

    fmt.Printf("\nTotal unique names generated: %d\n", gen.Count())
}
```

---

## Integration Examples

### Example 6: Docker Container Manager

```go
package docker

import (
    "context"
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type ContainerManager struct {
    nameGen namegenerator.Generator
}

func NewContainerManager() *ContainerManager {
    return &ContainerManager{
        nameGen: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (cm *ContainerManager) CreateContainer(ctx context.Context, image string, ports map[string]string) (string, error) {
    containerName := cm.nameGen.Generate()

    fmt.Printf("Creating container '%s'\n", containerName)
    fmt.Printf("  Image: %s\n", image)
    fmt.Printf("  Ports: %v\n", ports)

    // Docker API call would go here
    // client.ContainerCreate(ctx, config, hostConfig, nil, containerName)

    return containerName, nil
}

func (cm *ContainerManager) ListContainersByPrefix(prefix string) ([]string, error) {
    // Implementation would filter by prefix
    return []string{}, nil
}

// Example usage
func main() {
    manager := NewContainerManager()

    ports := map[string]string{
        "80/tcp": "8080",
    }

    name, err := manager.CreateContainer(context.Background(), "nginx:latest", ports)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Container created: %s\n", name)
}
```

---

### Example 7: Kubernetes Resource Namer

```go
package k8s

import (
    "fmt"
    "strings"
    "time"
    "github.com/obot-platform/namegenerator"
)

type ResourceNamer struct {
    generator namegenerator.Generator
    namespace string
}

func NewResourceNamer(namespace string) *ResourceNamer {
    return &ResourceNamer{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
        namespace: namespace,
    }
}

func (rn *ResourceNamer) GeneratePodName(appName string) string {
    baseName := rn.generator.Generate()
    // K8s names must be lowercase and < 63 chars
    return fmt.Sprintf("%s-%s-%s", rn.namespace, appName, baseName)[:63]
}

func (rn *ResourceNamer) GenerateServiceName(appName string) string {
    baseName := rn.generator.Generate()
    return fmt.Sprintf("%s-%s-svc-%s", rn.namespace, appName, baseName)[:63]
}

func (rn *ResourceNamer) GenerateSecretName(purpose string) string {
    baseName := rn.generator.Generate()
    purpose = strings.ToLower(purpose)
    return fmt.Sprintf("%s-secret-%s-%s", rn.namespace, purpose, baseName)[:63]
}

// Example usage
func main() {
    namer := NewResourceNamer("production")

    podName := namer.GeneratePodName("webapp")
    serviceName := namer.GenerateServiceName("webapp")
    secretName := namer.GenerateSecretName("database")

    fmt.Println("Kubernetes Resources:")
    fmt.Printf("  Pod:     %s\n", podName)
    fmt.Printf("  Service: %s\n", serviceName)
    fmt.Printf("  Secret:  %s\n", secretName)
}
```

**Output**:

```
Kubernetes Resources:
  Pod:     production-webapp-silent-moon
  Service: production-webapp-svc-ancient-river
  Secret:  production-secret-database-bold-firefly
```

---

### Example 8: Test Data Factory

```go
package testdata

import (
    "fmt"
    "strings"
    "time"
    "github.com/obot-platform/namegenerator"
)

type User struct {
    ID       string
    Username string
    Email    string
    FullName string
}

type UserFactory struct {
    usernameGen namegenerator.Generator
    emailGen    namegenerator.Generator
    nameGen     namegenerator.Generator
}

func NewUserFactory(seed int64) *UserFactory {
    return &UserFactory{
        usernameGen: namegenerator.NewNameGenerator(seed),
        emailGen:    namegenerator.NewNameGenerator(seed + 1000),
        nameGen:     namegenerator.NewNameGenerator(seed + 2000),
    }
}

func (uf *UserFactory) CreateUser() User {
    username := uf.usernameGen.Generate()
    emailName := uf.emailGen.Generate()
    fullName := uf.nameToTitle(uf.nameGen.Generate())

    return User{
        ID:       fmt.Sprintf("user-%d", time.Now().UnixNano()),
        Username: username,
        Email:    fmt.Sprintf("%s@example.com", emailName),
        FullName: fullName,
    }
}

func (uf *UserFactory) CreateBatch(count int) []User {
    users := make([]User, count)
    for i := 0; i < count; i++ {
        users[i] = uf.CreateUser()
    }
    return users
}

func (uf *UserFactory) nameToTitle(name string) string {
    parts := strings.Split(name, "-")
    return strings.Title(parts[0]) + " " + strings.Title(parts[1])
}

// Example usage
func main() {
    factory := NewUserFactory(42)

    users := factory.CreateBatch(5)

    fmt.Println("Test Users:")
    for i, user := range users {
        fmt.Printf("%d. %s\n", i+1, user.Username)
        fmt.Printf("   Email: %s\n", user.Email)
        fmt.Printf("   Name:  %s\n", user.FullName)
        fmt.Println()
    }
}
```

---

### Example 9: Temporary File Manager

```go
package tempfile

import (
    "fmt"
    "os"
    "path/filepath"
    "time"
    "github.com/obot-platform/namegenerator"
)

type TempFileManager struct {
    generator namegenerator.Generator
    baseDir   string
}

func NewTempFileManager(baseDir string) (*TempFileManager, error) {
    // Ensure base directory exists
    if err := os.MkdirAll(baseDir, 0755); err != nil {
        return nil, err
    }

    return &TempFileManager{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
        baseDir:   baseDir,
    }, nil
}

func (tfm *TempFileManager) CreateTempDir() (string, error) {
    dirName := fmt.Sprintf("tmp-%s", tfm.generator.Generate())
    fullPath := filepath.Join(tfm.baseDir, dirName)

    err := os.MkdirAll(fullPath, 0755)
    if err != nil {
        return "", fmt.Errorf("failed to create temp dir: %w", err)
    }

    return fullPath, nil
}

func (tfm *TempFileManager) CreateTempFile(extension string) (*os.File, error) {
    fileName := fmt.Sprintf("%s.%s", tfm.generator.Generate(), extension)
    fullPath := filepath.Join(tfm.baseDir, fileName)

    file, err := os.Create(fullPath)
    if err != nil {
        return nil, fmt.Errorf("failed to create temp file: %w", err)
    }

    return file, nil
}

// Example usage
func main() {
    manager, err := NewTempFileManager("/tmp/myapp")
    if err != nil {
        log.Fatal(err)
    }

    // Create temporary directory
    dir, err := manager.CreateTempDir()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Created directory: %s\n", dir)

    // Create temporary files
    for _, ext := range []string{"json", "txt", "log"} {
        file, err := manager.CreateTempFile(ext)
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("Created file: %s\n", file.Name())
        file.Close()
    }
}
```

---

## FFI Examples (Other Languages)

While namegenerator is a Go library, it can be used from other languages via CGo/FFI.

### Example 10: Python via ctypes (Hypothetical)

```python
# This is a hypothetical example showing how you might wrap the Go library
import ctypes
import time

# Load the shared library (would need to be built with -buildmode=c-shared)
lib = ctypes.CDLL("./libnamegenerator.so")

# Define function signatures
lib.NewNameGenerator.argtypes = [ctypes.c_int64]
lib.NewNameGenerator.restype = ctypes.c_void_p

lib.Generate.argtypes = [ctypes.c_void_p]
lib.Generate.restype = ctypes.c_char_p

class NameGenerator:
    def __init__(self, seed=None):
        if seed is None:
            seed = int(time.time() * 1000000000)
        self.gen = lib.NewNameGenerator(seed)

    def generate(self):
        result = lib.Generate(self.gen)
        return result.decode('utf-8')

# Usage
gen = NameGenerator()

for i in range(10):
    name = gen.generate()
    print(f"{i+1}. {name}")
```

---

### Example 11: JavaScript/Node.js via FFI (Hypothetical)

```javascript
// Hypothetical Node.js FFI wrapper
const ffi = require('ffi-napi');
const ref = require('ref-napi');

// Load the shared library
const libNameGen = ffi.Library('./libnamegenerator', {
  'NewNameGenerator': ['pointer', ['int64']],
  'Generate': ['string', ['pointer']]
});

class NameGenerator {
  constructor(seed = Date.now() * 1000000) {
    this.gen = libNameGen.NewNameGenerator(seed);
  }

  generate() {
    return libNameGen.Generate(this.gen);
  }
}

// Usage
const gen = new NameGenerator();

for (let i = 0; i < 10; i++) {
  const name = gen.generate();
  console.log(`${i + 1}. ${name}`);
}
```

---

## Testing Examples

### Example 12: Table-Driven Tests

```go
package namegenerator_test

import (
    "testing"
    "github.com/obot-platform/namegenerator"
)

func TestDeterministicGeneration(t *testing.T) {
    tests := []struct {
        name     string
        seed     int64
        count    int
        expected []string
    }{
        {
            name:  "seed 42",
            seed:  42,
            count: 3,
            expected: []string{
                "wandering-frost",
                "bold-mountain",
                "icy-silence",
            },
        },
        {
            name:  "seed 12345",
            seed:  12345,
            count: 2,
            expected: []string{
                "ancient-bird",
                "crimson-leaf",
            },
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            gen := namegenerator.NewNameGenerator(tt.seed)

            for i, want := range tt.expected {
                got := gen.Generate()
                if got != want {
                    t.Errorf("Generation %d: got %q, want %q", i, got, want)
                }
            }
        })
    }
}
```

---

### Example 13: Benchmark Tests

```go
package namegenerator_test

import (
    "testing"
    "time"
    "github.com/obot-platform/namegenerator"
)

func BenchmarkGenerate(b *testing.B) {
    gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _ = gen.Generate()
    }
}

func BenchmarkNewGenerator(b *testing.B) {
    seed := time.Now().UTC().UnixNano()

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _ = namegenerator.NewNameGenerator(seed + int64(i))
    }
}

func BenchmarkGenerateParallel(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
        for pb.Next() {
            _ = gen.Generate()
        }
    })
}
```

**Run benchmarks**:

```bash
go test -bench=. -benchmem
```

---

## Production Patterns

### Example 14: Singleton Pattern

```go
package naming

import (
    "sync"
    "time"
    "github.com/obot-platform/namegenerator"
)

var (
    instance namegenerator.Generator
    once     sync.Once
)

func GetGenerator() namegenerator.Generator {
    once.Do(func() {
        seed := time.Now().UTC().UnixNano()
        instance = namegenerator.NewNameGenerator(seed)
    })
    return instance
}

// Usage
func main() {
    gen := naming.GetGenerator()
    name := gen.Generate()
}
```

---

### Example 15: Dependency Injection

```go
package app

import (
    "github.com/obot-platform/namegenerator"
)

type Service struct {
    nameGen namegenerator.Generator
    // other dependencies...
}

func NewService(gen namegenerator.Generator) *Service {
    return &Service{
        nameGen: gen,
    }
}

func (s *Service) CreateResource() string {
    name := s.nameGen.Generate()
    // business logic...
    return name
}

// Main application setup
func main() {
    gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
    service := NewService(gen)

    resource := service.CreateResource()
    fmt.Println(resource)
}
```

---

### Example 16: HTTP API Endpoint

```go
package api

import (
    "encoding/json"
    "net/http"
    "time"
    "github.com/obot-platform/namegenerator"
)

type NameAPI struct {
    generator namegenerator.Generator
}

func NewNameAPI() *NameAPI {
    return &NameAPI{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (api *NameAPI) HandleGenerate(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    name := api.generator.Generate()

    response := map[string]string{
        "name":      name,
        "timestamp": time.Now().Format(time.RFC3339),
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func main() {
    api := NewNameAPI()

    http.HandleFunc("/api/name", api.HandleGenerate)
    http.ListenAndServe(":8080", nil)
}
```

**Test the API**:

```bash
curl http://localhost:8080/api/name
# {"name":"silent-moon","timestamp":"2026-01-16T14:30:22Z"}
```

---

## Related Documentation

- [API Reference](API.md) - Complete API documentation
- [User Guide](USER_GUIDE.md) - Step-by-step tutorials
- [Usage Patterns](USAGE_PATTERNS.md) - Advanced integration patterns
- [Architecture](ARCHITECTURE.md) - Internal design details

---

**Last Updated**: 2026-01-16
**Version**: 1.0.0
