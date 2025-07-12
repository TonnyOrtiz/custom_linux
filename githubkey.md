# How to generate the github key for ssh conection

```
# Go to your ssh directory
cd ~/ssh

# Generate the ssh key, it will ask for the file name
ssh-keygen -t ed25519 -C "tonny.ortiz.95@gmail.com" 

# Add the keys to 
ssh-add ~/.ssh/id_ed25519

# paste the .pub key to github
```

