# Varly

Variable storage and reading for shell scripts. Decouple variables/values/data from your bash scripts. Make your bash scripts safe to share and check in to source control without leaking data.

## How it works

Uses a yaml file to store groups of variables. The script is a thin wrapper using `yq` to read values from the yaml file.

At the moment the script only reads values from the yaml file so you must manually edit the yaml file to add or edit something in it.

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

## Configuration options

Set these environment variables to override defaults.

| Variable     | Description                                       | Defaults to                                                      |
| ------------ | ------------------------------------------------- | ---------------------------------------------------------------- |
| `VARLY_DIR`  | Path to the directory where `varly` is located.   | [Required] You must set this in your .zshrc, .bashrc or similar. |
| `VARLY_FILE` | Name of the yaml file where variables are stored. | `vars.yaml`                                                      |
| `VARLY_YQ`   | Path to the `yq` executable to use.               | `$VARLY_DIR/bin/yq`                                              |

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
