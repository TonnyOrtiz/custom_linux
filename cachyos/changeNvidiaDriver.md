# How to change Nvidia Driver


### Install Closed drivers
~~~
sudo pacman -S \
nvidia-580xx-dkms \
nvidia-580xx-utils \
lib32-nvidia-580xx-utils \
nvidia-580xx-settings \
opencl-nvidia-580xx \
lib32-opencl-nvidia-580xx
~~~


### Install Open drivers
```
sudo pacman -S linux-cachyos-nvidia-open linux-cachyos-lts-nvidia-open nvidia-utils lib32-nvidia-utils nvidia-settings opencl-nvidia lib32-opencl-nvidia                 
```

Si querés estabilidad absoluta

Después de instalar 580xx, podés lockear:

~~~
sudo nano /etc/pacman.conf
~~~

Y agregar:
~~~
IgnorePkg = nvidia-580xx-dkms nvidia-580xx-utils lib32-nvidia-580xx-utils nvidia-580xx-settings opencl-nvidia-580xx lib32-opencl-nvidia-580xx
~~~

