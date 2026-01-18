# Usage Patterns & Advanced Scenarios

This document provides advanced usage patterns, real-world scenarios, and best practices for the namegenerator library.

---

## Table of Contents

- [Common Use Cases](#common-use-cases)
- [Concurrent Usage Patterns](#concurrent-usage-patterns)
- [Testing Strategies](#testing-strategies)
- [Integration Patterns](#integration-patterns)
- [Custom Implementations](#custom-implementations)
- [Performance Optimization](#performance-optimization)
- [Error Handling & Edge Cases](#error-handling--edge-cases)
- [Best Practices](#best-practices)

---

## Common Use Cases

### 1. Docker Container Naming

```go
package main

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type ContainerManager struct {
    generator namegenerator.Generator
}

func NewContainerManager() *ContainerManager {
    return &ContainerManager{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (cm *ContainerManager) CreateContainer(image string) {
    containerName := cm.generator.Generate()
    fmt.Printf("Creating container '%s' from image '%s'\n", containerName, image)
    // Container creation logic here
}

func main() {
    manager := NewContainerManager()

    manager.CreateContainer("nginx:latest")
    // Output: Creating container 'silent-moon' from image 'nginx:latest'

    manager.CreateContainer("postgres:14")
    // Output: Creating container 'ancient-river' from image 'postgres:14'
}
```

---

### 2. Kubernetes Resource Naming

```go
package k8s

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type ResourceNamer struct {
    generator namegenerator.Generator
    prefix    string
}

func NewResourceNamer(prefix string) *ResourceNamer {
    return &ResourceNamer{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
        prefix:    prefix,
    }
}

func (rn *ResourceNamer) GeneratePodName() string {
    return fmt.Sprintf("%s-pod-%s", rn.prefix, rn.generator.Generate())
}

func (rn *ResourceNamer) GenerateServiceName() string {
    return fmt.Sprintf("%s-svc-%s", rn.prefix, rn.generator.Generate())
}

func main() {
    namer := NewResourceNamer("myapp")

    podName := namer.GeneratePodName()
    fmt.Println(podName) // myapp-pod-autumn-waterfall

    serviceName := namer.GenerateServiceName()
    fmt.Println(serviceName) // myapp-svc-silent-moon
}
```

---

### 3. Test Data Generation

```go
package testing

import (
    "testing"
    "github.com/obot-platform/namegenerator"
)

func TestUserCreation(t *testing.T) {
    gen := namegenerator.NewNameGenerator(42) // Fixed seed for reproducibility

    testCases := []struct {
        username string
        email    string
    }{
        {
            username: gen.Generate(),
            email:    gen.Generate() + "@example.com",
        },
        {
            username: gen.Generate(),
            email:    gen.Generate() + "@example.com",
        },
    }

    for _, tc := range testCases {
        t.Run(tc.username, func(t *testing.T) {
            user := CreateUser(tc.username, tc.email)
            if user == nil {
                t.Fatalf("Failed to create user: %s", tc.username)
            }
        })
    }
}
```

---

### 4. Temporary File/Directory Naming

```go
package storage

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

func NewTempFileManager(baseDir string) *TempFileManager {
    return &TempFileManager{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
        baseDir:   baseDir,
    }
}

func (tfm *TempFileManager) CreateTempDir() (string, error) {
    dirName := fmt.Sprintf("tmp-%s", tfm.generator.Generate())
    fullPath := filepath.Join(tfm.baseDir, dirName)

    err := os.MkdirAll(fullPath, 0755)
    if err != nil {
        return "", err
    }

    return fullPath, nil
}

func (tfm *TempFileManager) CreateTempFile(extension string) (string, error) {
    fileName := fmt.Sprintf("%s.%s", tfm.generator.Generate(), extension)
    fullPath := filepath.Join(tfm.baseDir, fileName)

    file, err := os.Create(fullPath)
    if err != nil {
        return "", err
    }
    defer file.Close()

    return fullPath, nil
}

func main() {
    manager := NewTempFileManager("/tmp")

    dir, _ := manager.CreateTempDir()
    fmt.Println(dir) // /tmp/tmp-silent-moon

    file, _ := manager.CreateTempFile("json")
    fmt.Println(file) // /tmp/ancient-river.json
}
```

---

### 5. Database Snapshot Naming

```go
package database

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type SnapshotManager struct {
    generator namegenerator.Generator
}

func NewSnapshotManager() *SnapshotManager {
    return &SnapshotManager{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (sm *SnapshotManager) CreateSnapshot(dbName string) string {
    timestamp := time.Now().Format("20060102-150405")
    randomName := sm.generator.Generate()

    snapshotName := fmt.Sprintf("%s-backup-%s-%s", dbName, timestamp, randomName)
    return snapshotName
}

func main() {
    manager := NewSnapshotManager()

    snapshot := manager.CreateSnapshot("production")
    fmt.Println(snapshot)
    // Output: production-backup-20260116-143022-silent-moon
}
```

---

## Concurrent Usage Patterns

### Pattern 1: Generator Pool

```go
package pool

import (
    "sync"
    "time"
    "github.com/obot-platform/namegenerator"
)

type GeneratorPool struct {
    generators chan namegenerator.Generator
}

func NewGeneratorPool(size int) *GeneratorPool {
    pool := &GeneratorPool{
        generators: make(chan namegenerator.Generator, size),
    }

    baseTime := time.Now().UTC().UnixNano()
    for i := 0; i < size; i++ {
        gen := namegenerator.NewNameGenerator(baseTime + int64(i))
        pool.generators <- gen
    }

    return pool
}

func (gp *GeneratorPool) Generate() string {
    gen := <-gp.generators
    name := gen.Generate()
    gp.generators <- gen
    return name
}

func main() {
    pool := NewGeneratorPool(10)

    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            name := pool.Generate()
            fmt.Printf("Worker %d: %s\n", id, name)
        }(i)
    }
    wg.Wait()
}
```

---

### Pattern 2: Per-Goroutine Generator

```go
package concurrent

import (
    "fmt"
    "sync"
    "time"
    "github.com/obot-platform/namegenerator"
)

type Worker struct {
    id        int
    generator namegenerator.Generator
}

func NewWorker(id int) *Worker {
    seed := time.Now().UTC().UnixNano() + int64(id*1000)
    return &Worker{
        id:        id,
        generator: namegenerator.NewNameGenerator(seed),
    }
}

func (w *Worker) ProcessBatch(count int) []string {
    names := make([]string, count)
    for i := 0; i < count; i++ {
        names[i] = w.generator.Generate()
    }
    return names
}

func main() {
    var wg sync.WaitGroup
    numWorkers := 5

    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(workerId int) {
            defer wg.Done()

            worker := NewWorker(workerId)
            names := worker.ProcessBatch(10)

            fmt.Printf("Worker %d generated: %v\n", workerId, names)
        }(i)
    }

    wg.Wait()
}
```

---

### Pattern 3: Thread-Safe Wrapper

```go
package safe

import (
    "sync"
    "time"
    "github.com/obot-platform/namegenerator"
)

type SafeGenerator struct {
    mu   sync.Mutex
    gen  namegenerator.Generator
}

func NewSafeGenerator() *SafeGenerator {
    return &SafeGenerator{
        gen: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (sg *SafeGenerator) Generate() string {
    sg.mu.Lock()
    defer sg.mu.Unlock()
    return sg.gen.Generate()
}

// Alternative: RWMutex for potential read optimizations
type SafeGeneratorRW struct {
    mu   sync.RWMutex
    gen  namegenerator.Generator
}

func NewSafeGeneratorRW() *SafeGeneratorRW {
    return &SafeGeneratorRW{
        gen: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (sg *SafeGeneratorRW) Generate() string {
    sg.mu.Lock()
    defer sg.mu.Unlock()
    return sg.gen.Generate()
}

func main() {
    safeGen := NewSafeGenerator()

    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            name := safeGen.Generate()
            fmt.Println(name)
        }()
    }
    wg.Wait()
}
```

---

## Testing Strategies

### Strategy 1: Deterministic Testing

```go
package mypackage_test

import (
    "testing"
    "github.com/obot-platform/namegenerator"
)

func TestDeterministicGeneration(t *testing.T) {
    // Fixed seed ensures reproducible test results
    gen := namegenerator.NewNameGenerator(12345)

    expected := []string{
        // Pre-computed expected values for seed 12345
        "wandering-frost",
        "bold-mountain",
        // ... add more expected values
    }

    for i, want := range expected {
        got := gen.Generate()
        if got != want {
            t.Errorf("Generation %d: got %q, want %q", i, got, want)
        }
    }
}
```

---

### Strategy 2: Format Validation Testing

```go
package mypackage_test

import (
    "regexp"
    "testing"
    "time"
    "github.com/obot-platform/namegenerator"
)

func TestNameFormat(t *testing.T) {
    gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())

    // Pattern: word-word (no numbers, special chars)
    pattern := regexp.MustCompile(`^[a-z]+-[a-z]+$`)

    for i := 0; i < 1000; i++ {
        name := gen.Generate()

        if !pattern.MatchString(name) {
            t.Errorf("Invalid name format: %q (iteration %d)", name, i)
        }

        if len(name) < 3 || len(name) > 50 {
            t.Errorf("Invalid name length: %q (len=%d)", name, len(name))
        }
    }
}
```

---

### Strategy 3: Uniqueness Testing (Statistical)

```go
package mypackage_test

import (
    "testing"
    "time"
    "github.com/obot-platform/namegenerator"
)

func TestUniquenessRate(t *testing.T) {
    gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())

    iterations := 10000
    seen := make(map[string]int)

    for i := 0; i < iterations; i++ {
        name := gen.Generate()
        seen[name]++
    }

    uniqueCount := len(seen)
    uniqueRate := float64(uniqueCount) / float64(iterations) * 100

    t.Logf("Unique names: %d/%d (%.2f%%)", uniqueCount, iterations, uniqueRate)

    // Expect at least 95% uniqueness over 10k iterations
    // Theoretical: 3,596 possible combinations
    if uniqueRate < 95.0 {
        t.Errorf("Uniqueness rate too low: %.2f%% (expected >= 95%%)", uniqueRate)
    }

    // Find duplicates
    for name, count := range seen {
        if count > 5 {
            t.Logf("Name %q appeared %d times", name, count)
        }
    }
}
```

---

### Strategy 4: Benchmark Testing

```go
package mypackage_test

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

func BenchmarkGenerateConcurrent(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
        for pb.Next() {
            _ = gen.Generate()
        }
    })
}
```

---

## Integration Patterns

### Pattern 1: HTTP Service Integration

```go
package server

import (
    "encoding/json"
    "net/http"
    "time"
    "github.com/obot-platform/namegenerator"
)

type NameService struct {
    generator namegenerator.Generator
}

func NewNameService() *NameService {
    return &NameService{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (ns *NameService) GenerateNameHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    name := ns.generator.Generate()

    response := map[string]string{"name": name}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func main() {
    service := NewNameService()
    http.HandleFunc("/api/name", service.GenerateNameHandler)
    http.ListenAndServe(":8080", nil)
}
```

---

### Pattern 2: CLI Tool Integration

```go
package main

import (
    "flag"
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

func main() {
    count := flag.Int("count", 1, "Number of names to generate")
    seed := flag.Int64("seed", 0, "Random seed (0 for current time)")
    prefix := flag.String("prefix", "", "Prefix for generated names")
    flag.Parse()

    // Use current time if seed not provided
    if *seed == 0 {
        *seed = time.Now().UTC().UnixNano()
    }

    gen := namegenerator.NewNameGenerator(*seed)

    for i := 0; i < *count; i++ {
        name := gen.Generate()
        if *prefix != "" {
            name = fmt.Sprintf("%s-%s", *prefix, name)
        }
        fmt.Println(name)
    }
}

// Usage:
// $ go run cli.go --count 5 --prefix myapp
// myapp-silent-moon
// myapp-ancient-river
// myapp-bold-firefly
// myapp-withered-leaf
// myapp-crimson-mountain
```

---

### Pattern 3: Database Integration

```go
package database

import (
    "database/sql"
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type ResourceManager struct {
    db        *sql.DB
    generator namegenerator.Generator
}

func NewResourceManager(db *sql.DB) *ResourceManager {
    return &ResourceManager{
        db:        db,
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (rm *ResourceManager) CreateResourceWithUniqueName(resourceType string) (string, error) {
    maxAttempts := 10

    for attempt := 0; attempt < maxAttempts; attempt++ {
        name := rm.generator.Generate()

        // Check if name exists
        var exists bool
        err := rm.db.QueryRow(
            "SELECT EXISTS(SELECT 1 FROM resources WHERE name = $1)",
            name,
        ).Scan(&exists)

        if err != nil {
            return "", err
        }

        if !exists {
            // Name is unique, insert resource
            _, err := rm.db.Exec(
                "INSERT INTO resources (name, type, created_at) VALUES ($1, $2, $3)",
                name, resourceType, time.Now(),
            )
            if err != nil {
                return "", err
            }
            return name, nil
        }

        // Name collision, try again
    }

    return "", fmt.Errorf("failed to generate unique name after %d attempts", maxAttempts)
}
```

---

## Custom Implementations

### Custom Word Lists

```go
package custom

import (
    "fmt"
    "math/rand"
)

// CustomGenerator with custom word lists
type CustomGenerator struct {
    random     *rand.Rand
    adjectives []string
    nouns      []string
}

func NewCustomGenerator(seed int64, adjectives, nouns []string) *CustomGenerator {
    return &CustomGenerator{
        random:     rand.New(rand.NewSource(seed)),
        adjectives: adjectives,
        nouns:      nouns,
    }
}

func (cg *CustomGenerator) Generate() string {
    adj := cg.adjectives[cg.random.Intn(len(cg.adjectives))]
    noun := cg.nouns[cg.random.Intn(len(cg.nouns))]
    return fmt.Sprintf("%s-%s", adj, noun)
}

func main() {
    // Technical-themed word lists
    adjectives := []string{"fast", "secure", "scalable", "robust", "efficient"}
    nouns := []string{"server", "database", "cache", "proxy", "gateway"}

    gen := NewCustomGenerator(42, adjectives, nouns)
    fmt.Println(gen.Generate()) // fast-server
}
```

---

### Multi-Part Names

```go
package multipart

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type MultiPartGenerator struct {
    gen1 namegenerator.Generator
    gen2 namegenerator.Generator
}

func NewMultiPartGenerator() *MultiPartGenerator {
    seed := time.Now().UTC().UnixNano()
    return &MultiPartGenerator{
        gen1: namegenerator.NewNameGenerator(seed),
        gen2: namegenerator.NewNameGenerator(seed + 1000),
    }
}

func (mpg *MultiPartGenerator) GenerateLongName() string {
    part1 := mpg.gen1.Generate()
    part2 := mpg.gen2.Generate()
    return fmt.Sprintf("%s-%s", part1, part2)
}

func main() {
    gen := NewMultiPartGenerator()
    fmt.Println(gen.GenerateLongName())
    // Output: silent-moon-ancient-river
}
```

---

## Performance Optimization

### Pre-allocation Strategy

```go
package optimized

import (
    "time"
    "github.com/obot-platform/namegenerator"
)

func GenerateBatch(count int) []string {
    gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())

    // Pre-allocate slice to avoid reallocation
    names := make([]string, 0, count)

    for i := 0; i < count; i++ {
        names = append(names, gen.Generate())
    }

    return names
}
```

---

### Generator Reuse

```go
package reuse

import (
    "time"
    "github.com/obot-platform/namegenerator"
)

// Good: Reuse generator instance
type Service struct {
    generator namegenerator.Generator
}

func NewService() *Service {
    return &Service{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
    }
}

func (s *Service) CreateResource() string {
    return s.generator.Generate()
}

// Bad: Creating new generator each time
func CreateResourceBad() string {
    gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
    return gen.Generate()
}
```

---

## Error Handling & Edge Cases

### Handling Collisions

```go
package collision

import (
    "fmt"
    "time"
    "github.com/obot-platform/namegenerator"
)

type UniqueNameGenerator struct {
    generator namegenerator.Generator
    used      map[string]bool
}

func NewUniqueNameGenerator() *UniqueNameGenerator {
    return &UniqueNameGenerator{
        generator: namegenerator.NewNameGenerator(time.Now().UTC().UnixNano()),
        used:      make(map[string]bool),
    }
}

func (ung *UniqueNameGenerator) Generate() (string, error) {
    maxAttempts := 100

    for attempt := 0; attempt < maxAttempts; attempt++ {
        name := ung.generator.Generate()

        if !ung.used[name] {
            ung.used[name] = true
            return name, nil
        }
    }

    return "", fmt.Errorf("failed to generate unique name after %d attempts", maxAttempts)
}
```

---

## Best Practices

### ✅ Do's

1. **Reuse Generator Instances**

   ```go
   // Good
   type App struct {
       nameGen namegenerator.Generator
   }
   ```

2. **Use Time-Based Seeds for Production**

   ```go
   gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
   ```

3. **Use Fixed Seeds for Testing**

   ```go
   gen := namegenerator.NewNameGenerator(12345)
   ```

4. **Create Separate Generators for Goroutines**

   ```go
   go func() {
       gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
       // Use gen in goroutine
   }()
   ```

5. **Add Context-Specific Prefixes/Suffixes**

   ```go
   name := fmt.Sprintf("prod-%s", gen.Generate())
   ```

### ❌ Don'ts

1. **Don't Share Generators Across Goroutines**

   ```go
   // Bad: Race condition
   gen := namegenerator.NewNameGenerator(seed)
   go func() { gen.Generate() }()
   go func() { gen.Generate() }()
   ```

2. **Don't Create New Generators in Hot Paths**

   ```go
   // Bad: Performance impact
   for i := 0; i < 10000; i++ {
       gen := namegenerator.NewNameGenerator(time.Now().UTC().UnixNano())
       name := gen.Generate()
   }
   ```

3. **Don't Assume Uniqueness Without Validation**

   ```go
   // Bad: May have collisions
   names := make([]string, 5000)
   for i := range names {
       names[i] = gen.Generate() // No uniqueness check
   }
   ```

4. **Don't Use Same Seed for Different Purposes**

   ```go
   // Bad: Predictable sequences
   seed := 12345
   gen1 := namegenerator.NewNameGenerator(seed)
   gen2 := namegenerator.NewNameGenerator(seed) // Same sequence!
   ```

---

**Related Documentation:**

- [API Reference](API.md)
- [Project Index](INDEX.md)
- [Architecture](ARCHITECTURE.md)

---

**Last Updated**: 2026-01-16
