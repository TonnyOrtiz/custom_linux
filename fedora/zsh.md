### 1️⃣ Install Zsh

```bash
sudo dnf install zsh -y
```

Verify installation:

```bash
zsh --version
```

Set Zsh as your default shell:

```bash
chsh -s $(which zsh)
```

Log out and log back in afterward.

### 2️⃣ Install Oh My Zsh

Install git and curl if you don’t have them:

```bash
sudo dnf install git curl -y
```

Run the official installer:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

This creates:

```bash
~/.oh-my-zsh
~/.zshrc
```

### 3️⃣ Install Powerlevel10k
Clone the theme into Oh My Zsh’s custom themes directory:

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Edit your ~/.zshrc and change the theme line:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Then reload:

```bash
source ~/.zshrc
```

### 4️⃣ Install Recommended Fonts (Very Important)
Powerlevel10k requires a Nerd Font.

Install Nerd Fonts on Fedora:

```bash
sudo dnf install fira-code-fonts
```

For full icon support (recommended), install a Nerd Font manually:

Download from: https://www.nerdfonts.com/font-downloads

Extract to:

```bash
~/.local/share/fonts
```

Refresh font cache:

```bash
fc-cache -fv
```

Then set your terminal font to a Nerd Font (e.g., FiraCode Nerd Font).

If you're using WezTerm, add this to ~/.wezterm.lua:

```lua
local wezterm = require 'wezterm'

return {
  font = wezterm.font("FiraCode Nerd Font"),
}
```

### 5️⃣ Run Powerlevel10k Configuration
Restart your terminal or run:

```bash
p10k configure
```

Follow the interactive wizard.

### Optional (Recommended Plugins)
Edit ~/.zshrc and set:

```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

Then install them:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

Reload:

```bash
source ~/.zshrc
```


