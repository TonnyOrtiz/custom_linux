# Configuraciones de Zsh

Instalar zsh, oh my zsh y powerlevel10k

### Configurar powerlevel10k
```
p10k configure
```

### Instalación de plugins
```
git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search   
```

Agregar a .zshrc
```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-history-substring-search)   
```

Recargar configuración
```
source ~/.zshrc   
```

Actualizar los plugins con el siguiente oneliner
```
for plugin in ~/.oh-my-zsh/custom/plugins/*; do [[ -d "$plugin/.git" ]] && echo "Actualizando $plugin" && git -C "$plugin" pull; done   
```