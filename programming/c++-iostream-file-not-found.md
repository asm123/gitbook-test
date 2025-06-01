# C++: iostream file not found

* OS: MacOS Sequoia 15.3
* `which g++`: `/usr/bin/g++`
* Error: `fatal error: 'iostream' file not found`

`iostream` is a standard library in C++. If terminal can find `g++` command, but `g++` cannot find the `iostream` library, a likely culprit is an incomplete install of g++.

One of the possible solutions is to reinstall the command line tools.

```shell
sudo rm -rf /Library/Developer/CommandLineTools/
xcode-select --install
```

Now `g++ <code.cc>` produces the `a.out` executable.

Source: [this comment](https://github.com/ggml-org/llama.cpp/issues/9575#issuecomment-2412840553) on GitHub.
