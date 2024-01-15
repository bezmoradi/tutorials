# Go > Install A Program

-   In order to install a Go program, from inside the project folder run `go install`. This command will build and install the program in `GOPATH`. To what where the path for `GOPATH` is, run `go env` which outputs:

```text
.
.
.
GOPATH='/Users/behzadmoradi/go'
.
.
.
```

-   If cd into `/Users/behzadmoradi/go/bin`, you will see the name of the newly-installed program. From this point on, from everywhere on you file system you should be able to enter the name of the program and run it. If the `go install` command placed the executable in `/Users/behzadmoradi/go/bin` and you're unable to run it from anywhere without providing the full path, it might be because the `/Users/behzadmoradi/go/bin` directory is not added to your system's `PATH`. To check if this directory is in your `PATH`, you can open a terminal and type `echo $PATH`. If the output doesn't include `/Users/behzadmoradi/go/bin`, you'll need to add it. You can temporarily add it to your current session's `PATH` by running

```text
export PATH=$PATH:/Users/behzadmoradi/go/bin
```

-   To make this change persistent, so you won't have to run the command every time you open a terminal, you can add this line to your shell configuration file (`~/.bashrc`, `~/.bash_profile`, `~/.zshrc`, etc., depending on your shell). Edit the appropriate file and add the line:

```text
export PATH=$PATH:/Users/behzadmoradi/go/bin
```

-   After saving the changes and restarting your terminal or running `source` on the file to apply the changes (`source ~/.zshrc`), you should be able to run your `<program_name>` executable from anywhere without providing the full path.
