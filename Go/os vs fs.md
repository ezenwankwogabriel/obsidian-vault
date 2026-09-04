os - talks to the OS filestystem. to be used when you're reading/writing real files on dis
`os.Open, os.ReadDir, os.Stat, os.WriteFile, ...` 
io/fs - defines an abstract filesystem interface (fs.FS), not tied to disk. for writing code that works against any filesystem-like source: a real disk directory, zip archive, ... 
ex: `fs.ReadDir(fsys, path), fs.WalkDir(fsys, root, fn), fs.ReadFile(fsys, path)`. it takes `fs.FS` as their first argument instead of consuming the OS

In summary:
- Just reading real files on disk -> use `os` directly
- writing a function that should work on disk or on embedeed/test/mock data -> accept an `fs.FS` parameter and use `fs.ReadDir/fs.WalkDir`, and pass `os.DirFS(path)` at the call site

So except you're injecting a filesystem, use `os`. else `fs.FS`.
