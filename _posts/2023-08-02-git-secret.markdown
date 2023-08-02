---
published: true
title: git secret
layout: post
---

Generate a RSA key-pair:
```
gpg --gen-key
```

To export your public key, run:
```
gpg --armor --export your.email@address.com > ~/public-key.gpg
```

To import the public key of someone else (to share the secret with them for instance), run:
```
gpg --import public-key.gpg
```

Use your gpg key across different machines:
```
gpg --export-secret-key -a > ~/secretkey.asc
scp ~/secretkey.asc othermachine:~/
rm ~/secretkey.asc
ssh othermachine
gpg --import ~/secretkey.asc
rm ~/secretkey.asc
```

Installing git-secret:
```
brew install git-secret

# OR

cd ~/
git clone https://github.com/sobolevn/git-secret.git
cd git-secret
make
chmod +x git-secret
echo "export PATH=$(pwd):\${PATH}" >> ~/.bash_profile 
```

Initiate git-secret on a repo:
```
cd ~/my_repo
git-secret init
```

add yourself as a user with access (`-m` uses your current `git config user.email` setting as an identifier for the key ):
```
git-secret tell -m
```

Prevent a file from being pushed without encryption
```
echo cloud.config > .gitignore
```

Add a file to the list of encrypted files:
```
git-secret add cloud.config
```

Hide the files / encrypt:
```
git-secret hide
```

Reveal files / decrypt:
```
git-secret reveal 
```

Add other emails/users to the secrets:
```
git-secret tell <email1@mail.com>,<email2@mai.com>,<..>
git-secret hide
```

Push your changes:
```
git add -A . && git commit -m "updated git-secret" && git push
```

If you make changes to the revealed secreate you will need to hide it again before pushing so that the changes get stored in the encrypted file.
