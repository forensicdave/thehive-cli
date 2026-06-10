# Shell Completion for thehive-cli

The thehive-cli script now supports shell tab completion using `shtab`.

## Installation

### 1. Install shtab

```bash
pip install shtab
```

### 2. Generate and Install Completion Scripts

#### For Bash

```bash
# Create completion directory if it doesn't exist
mkdir -p ~/.local/share/bash-completion/completions

# Generate and install completion script
./thehive-cli --print-completion bash > ~/.local/share/bash-completion/completions/thehive-cli

# Reload your shell or source the file
source ~/.local/share/bash-completion/completions/thehive-cli
```

#### For Zsh

```bash
# Create completion directory if it doesn't exist
mkdir -p ~/.zsh/completion

# Generate and install completion script
./thehive-cli --print-completion zsh > ~/.zsh/completion/_thehive-cli

# Add to your ~/.zshrc (if not already there)
echo 'fpath=(~/.zsh/completion $fpath)' >> ~/.zshrc
echo 'autoload -Uz compinit && compinit' >> ~/.zshrc

# Reload your shell
source ~/.zshrc
```

#### For Fish

```bash
# Fish completion directory
mkdir -p ~/.config/fish/completions

# Generate and install completion script
./thehive-cli --print-completion fish > ~/.config/fish/completions/thehive-cli.fish
```

## Usage

Once installed, you can use tab completion with the thehive-cli command:

```bash
# Tab complete after typing the command
./thehive-cli --<TAB>

# Tab complete option values (for some options)
./thehive-cli --status <TAB>

# Complete file paths for --add-attachment
./thehive-cli -t 123 --add-attachment <TAB>
```

## Available Completions

The completion will suggest:
- All command-line options (--ticket, --list-all, --comments, etc.)
- Short options (-t, -v, etc.)
- File paths for --add-attachment
- Common values for some options

## Troubleshooting

### Completion not working

1. Make sure shtab is installed: `pip list | grep shtab`
2. Verify the completion script was generated correctly
3. Check that your shell's completion system is enabled
4. Try reloading your shell or starting a new terminal session

### Bash completion not loading

```bash
# Check if bash-completion is installed
# On macOS with Homebrew:
brew install bash-completion@2

# Add to your ~/.bash_profile or ~/.bashrc:
[[ -r "/opt/homebrew/etc/profile.d/bash_completion.sh" ]] && . "/opt/homebrew/etc/profile.d/bash_completion.sh"
```

### Zsh completion not loading

```bash
# Make sure these lines are in your ~/.zshrc:
fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit
```

## Updating Completions

After updating thehive-cli with new options, regenerate the completion script:

```bash
# For bash
./thehive-cli --print-completion bash > ~/.local/share/bash-completion/completions/thehive-cli

# For zsh
./thehive-cli --print-completion zsh > ~/.zsh/completion/_thehive-cli

# Reload your shell
```
