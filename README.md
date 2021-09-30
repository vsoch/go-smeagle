# Go Smeagle

This is mostly for fun - I wanted to see how hard it would be to parse a binary
with Go. Right now we use the same logic as objdump to load it, and then print
Symbols (and I found an entry to where the Dwarf is).

🚧️ **under development** 🚧️

## Usage

To run and preview the output, do:

```bash
$ make
$ go run main.go parse libtest.so
```
```
{"library":"libtest.so","functions":[{"name":"__printf_chk"},{"parameters":[{"name":"a","type":"long int","sizes":8},{"name":"b","type":"long int","sizes":8},{"name":"c","type":"long int","sizes":8},{"name":"d","type":"long int","sizes":8},{"name":"e","type":"long int","sizes":8},{"name":"f","type":"__int128","sizes":16}],"name":"bigcall"}]}
```

or print pretty:

```
$ go run main.go parse libtest.so --pretty
```

Note that this library is under development, so stay tuned!

## Background

I started this library after discussion (see [this thread](https://twitter.com/vsoch/status/1437535961131352065)) and wanting to extend Dwarf a bit and also reproduce [Smeagle](https://github.com/buildsi/Smeagle) in Go.

## Includes

Since I needed some functionality from [debug/dwarf](https://cs.opensource.google/go/go/+/master:src/debug/dwarf/) that was not public, the library is included here (with proper license/credit headers) in [pkg/debug](pkg/debug) along with ELF that needs to return a matching type. The changes I made include

 - renaming readType to ReadType so it's public.
 - also renaming sigToType to SigToType so it's public
 - made typeCache public (TypeCache)
 - Added an "Original" (interface) to a CommonType, and then changed ReadType in [dwarf/debug/type.go](pkg/dwarf/debug/type.go) so that each case sets `t.Original = t` so we can return the original type to further parse (`t.Common().Original`).
 - Added a StructCache to the dwarf.Data in [pkg/debug/dwarf/open.go](pkg/debub/dwarf/open.go) that is populated in [pkg/debug/dwarf/type.go](pkg/debug/dwarf/type.go) as follows:
 
```
// ADDED: save the struct to the struct cache for later lookup
d.StructCache[t.StructName] = t
```

And then used in [parsers/x86_64/parse.go](parsers/x86_64/parse.go) to match a typedef (which only has name and type string) to a fully parsed struct (a struct, union, or class).

 
## TODO

 - add variable parsing
 - add allocator to get allocations
 - need to get registers / locations for each type
  
