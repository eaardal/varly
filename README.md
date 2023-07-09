# Varly

Variable storage and reading for shell scripts. Decouple variables/values/data from your bash scripts. Make your bash scripts safe to share and check in to source control without leaking data.

## How it works

Uses a yaml file to store groups of variables. The script is a thin wrapper using `yq` to read values from the yaml file.

At the moment the script only reads values from the yaml file so you must manually edit the yaml file to add or edit something in it.

## How to get started

1. Put the `varly` bash script in this repo in a directory that's included in your PATH.
2. Add `export VARLY_DIR=your/path/to/varly/dir` In your .zshrc, .bashrc or similar.
3. Either put the `bin` directory with `yq` in this repo as a sub-directory to wherever you placed the `varly` script, or set `VARLY_YQ` to your own version of yq.

See [Configuration options](#configuration-options) for further (optional) setup.

## How to use

Given this yaml:

```yaml
# vars.yaml

hello: &hello-section
  world: &hello-world hi there
  mom: hi mom
more:
  values:
    etc: more values goes here
    greeting: *hello-world
  hello_again:
    <<: *hello-section
```

Read variables like this:

```shell
varly hello.world
# hi there

varly more.values.greeting
# hi there

varly more.hello_again.mom
# hi mom
```

### Sekret integration

There is a tiny integration with the sibling project [Sekret](https://github.com/eaardal/sekret).
If a value in `vars.yaml` is a Sekret command (such as `sekret get mysecret`), varly will evaluate (run) the command instead of just printing its value as is.

This obviously requires `sekret` to be on your PATH to work. If `sekret` is not installed, varly will print the command as is.

Example:

Using Sekret, the value of some_api.password resolves to "123456":

```shell
sekret get some_api.password
# 123456
```

In vars.yaml, insert the Sekret command as the password:

```yaml
some_api:
  username: myself
  password: sekret get some_api.password
```

Querying the password using varly will run the Sekret command and print its output:

```shell
varly some_api.password
# 123456
```

The first word of the variable value must be "sekret" for this to work.

## Configuration options

Set these environment variables to override defaults.

| Variable     | Description                                       | Defaults to                                                                    |
| ------------ | ------------------------------------------------- | ------------------------------------------------------------------------------ |
| `VARLY_DIR`  | Path to the directory where `varly` is located.   | :exclamation: [Required] You must set this in your .zshrc, .bashrc or similar. |
| `VARLY_FILE` | Name of the yaml file where variables are stored. | `vars.yaml`                                                                    |
| `VARLY_YQ`   | Path to the `yq` executable to use.               | `$VARLY_DIR/bin/yq`                                                            |

# YAML tips

## Re-use sections with references

- Use `&variable-name` to name something.
- Use `*variable-name` to reference something.
- Combine several references by using a list:

```yaml
something:
  <<:
    - *first-var
    - *second-var
```

Full references example:

```yaml
hello: &hello-section
  world: &hello-world hi there
  mom: hi mom
more:
  values:
    etc: more values goes here
    greeting: *hello-world
  hello_again:
    <<: *hello-section
```

Produces the computed yaml:

```yaml
hello:
  world: hi there
  mom: hi mom
more:
  values:
    etc: more values goes here
    greeting: hi there
  hello_again:
    world: hi there
    mom: hi mom
```
