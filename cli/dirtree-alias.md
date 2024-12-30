### Alias Command `dirtree`  for Directory Tree Visualization



The `dirtree` command is a useful tool for visualizing directory structures in a tree-like format, friendly for docs and LLMs consumption\
I introduce a version that:

1. Includes the current directory as the root of the tree.
2. Accepts a numerical parameter to limit the depth of the directory tree displayed.
3. Ignores dot files, dot directories, and files listed in `.gitignore`.

---

#### Updated Alias Command

Here is the updated alias command:

```bash
alias dirtree='function _dirtree() {
  local depth=${1:-999}
  local prefix="   "
  find . -type d ! -path "*/\.*" -maxdepth "$depth" 2>/dev/null | 
    sed -e "s|[^/]*/|--|g" -e "s|^|$prefix|" -e "s|--/|-/|g"
}; _dirtree'
```

This command exclude dot files, dot directories, and respecting `.gitignore` entries.
It processes the output with `sed` to format it in a tree-like style.

---

#### How to Add the Alias to Your Shell Configuration

1. **Edit Your Shell Configuration File**: Open your `~/.bashrc` or `~/.zshrc` file in a text editor.

   ```bash
   # or Vim/Emacs
   nano ~/.bashrc
   # or for zsh
   nano ~/.zshrc
   ```

2. **Add the Alias**: Copy and paste the alias command into the file.

3. **Reload the Configuration**: Save the file and reload your shell configuration by running:

   ```bash
   source ~/.bashrc
   # or for zsh
   source ~/.zshrc
   ```

---

#### Usage Examples

1. **Default Usage**: Display the full directory tree from the current directory, ignoring dot files, dot directories, and respecting `.gitignore`.

   ```bash
   dirtree
   ```

   Output:

   ```
      |
      |--dir1
      |----subdir1
      |----subdir2
      |--dir2
   ```

2. **Limit Depth**: Display a directory tree up to a specific depth.

   ```bash
   dirtree 2
   ```

   Output:

   ```
      |
      |--dir1
      |----subdir1
      |--dir2
   ```

   In this example, only directories up to two levels deep are shown.

---

#### Notes

- The default behavior (`dirtree` without arguments) lists all directories recursively while ignoring dot files and respecting `.gitignore`.
- This command works on Unix-like systems with `bash` or `zsh`.
- Make sure to initialize a Git repository (`git init`) if you want `.gitignore` to be considered.

---

Feel free to customize and extend this alias to suit your workflow!
