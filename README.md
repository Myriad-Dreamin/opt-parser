# opt-parser
simple scalable option parser

## Requirement

+ c++ minimum version: c++11
+ fmt library (will remove later)

## Example
```cpp

#include <opt/opt.h>

#define VERSION "v0.1.0"

std::string basename(std::string s) {
    for (size_t i = s.length() - 1; i >= 0; i--) {
        if (s[i] == '\\' || s[i] == '/') {
            return s.substr(i + 1);
        }
    }
    return s;
}

int main(int argc, const char *argv[]) {
    bool help = false;
    bool version = false;

    minimum::addOpts("help", &help, "Print help message", cat_default, false, "h");
    minimum::addOpts("version", &version, "Print Version", cat_default, false, "");
    minimum::parseOpts(argc, argv);

    if (help) {
        std::cout << "usage: " << basename(argv[0]) << " [--version] [-h | --help]\n\n";

        std::cout << minimum::print_helps();
        exit(0);
    }
}
```

## API Spec

### OptParser

methods:

##### add String Option
```cpp
void addOpts(StringRef long_opt, std::string *storage,
             StringRef desc = "", StringRef category = "", const char *default_value = "",
             StringRef short_opt = "");
```

##### add Int64Opt Option
```cpp
void addOpts(StringRef long_opt, int64_t *storage,
             StringRef desc = "", StringRef category = "", int64_t default_value = 0,
             StringRef short_opt = "");
```

##### add Boolean Option
```cpp
void addOpts(StringRef long_opt, bool *storage,
             StringRef desc = "", StringRef category = "", bool default_value = false,
             StringRef short_opt = "");
```

##### add User-defined Option

the option interface is a callback function
```cpp
using opt_has_value = bool;
using OptMap = std::map<std::string, std::pair<std::string, opt_has_value>>;
using ParseCallback = std::function<void(OptMap &opts)>;
void addOpts(const ParseCallback &callback);
```

### Advanced Usage

##### example of user-defined option

```cpp
// Functor Class
class StringOptPanicIfHasNoValue : public Opt {
    std::string *storage;
    const char *default_value;
public:
    StringOptPanicIfHasNoValue() noexcept:
            Opt("", "", "", ""), storage(nullptr), default_value("") {}

    StringOptPanicIfHasNoValue(StringRef long_opt, std::string *storage,
              StringRef desc = "", StringRef category = "", const char *default_value = "",
              StringRef short_opt = "")
            : Opt(long_opt, desc, short_opt, category), storage(storage), default_value(default_value) {
    }

    void operator()(OptMap &opts) {
        if (storage != nullptr) {
            Opt::operator()(opts, [this](auto parsed_value) {
                // if option parsed
                *storage = parsed_value.first;
            }, [this]() {
                // throw if not parsed
                throw std::invalid_argument(
                        std::string("expected option ") + long_opt);
            });
        }
    }
};
```

##### use my option builder

in cmake file:

```cmake
add_definitions(-DUSE_MY_BUILD_OPT_MAP)
```

and implement build option map function:

```cpp
void buildMap(OptMap &map, int &argc, const char **argv) {
   // ... from argv[argc] to map
}
```

##### define string heap size

each heap string pointer takes up 8 (or 4) bytes of memory or maybe have an union of 20 bytes small string

```cpp
#ifndef MAX_STRING_HEAP_SIZE
// default usage: 128KB/64KB ~ 320KB (sso)
#define MAX_STRING_HEAP_SIZE 1024 * 16
#endif
static std::string string_heap[MAX_STRING_HEAP_SIZE];
```