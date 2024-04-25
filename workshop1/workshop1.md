# Workshop 1

## Før workshop

- Installer git dersom du av en aller annen grunn ikke har gjort det
- Anbefaler at du på windows installerer windows terminal med oh my posh

https://learn.microsoft.com/en-us/windows/terminal/tutorials/custom-prompt-setup

## Workshop

### Lag en ny katalog på disk for workshopen

```shell
mkdir c:\git-workshop
mkdir c:\git-workshop\backup
mkdir c:\git-workshop\test
mkdir c:\git-workshop\worktree
cd c:\git-workshop
```

### Sett opp en config som bare gjelder for det filområdet. Kan være nyttig dersom du bytter mellom prosjekter som har flere repoer og du trenger egen config

```shell
notepad .gitconfig
```
Legg inn

```ini
[user]
  name = Mitt Navn
  email = min.epost@example.com
[init]
	defaultbranch = main
```

Editer git config global og legg en regel om at alle repoer under pathen c:\git-workshop skal bruke denne configen

```shell
git config --global -e
```

Legg til

```ini
[includeIf "gitdir:C:/git-workshop/"]
    path = C:/git-workshop/.gitconfig
```

### Lag et git repo og commmit noen endringer

```shell
cd c:\git-workshop\test
git init
echo "first" >> file.txt
git add file.txt
git commit -m "my first commit"
```

For å se alle commits som er gjort kan du bruke `git log`

```shell
git log
```

For å se mer detaljerte endringer kan du bruke `git show`

```shell
git log
```

Prøv også ut

```shell
git diff
```

### Lag et par brancher og se hva som skjer dersom vi lager en konflikt

```shell
git switch -c a
echo "some text" >> conflict.txt
git add conflict.txt
git commit -m "commit from branch a"

git switch -c b
echo "more text" >> conflict.txt
git add conflict.txt
git commit -m "commit from branch b"


git switch main
git merge a
git status
git log

git merge b
```

Nå vil merge feile med en merge conflict. La oss avbryte og prøve med en rebase.

```shell
git merge --abort

git switch b
git rebase main
type conflict.txt
```

Klarer ikke å rebase når det er konflikt heller så vi må fikse før vi kan fortsette.

```shell
echo "some text" > conflict.txt
echo "more text" >> conflict.txt
git rebase --continue

git switch main
git merge b
```

### Lag et bare repository og sett den opp som remote

Bare repository kan være nyttig når du trenger å dele kode med noen uten at du har tilgang på et repository i skyen.
Har selv brukt det til å synke noen code snippets mellom maskiner.

```shell
cd c:\git-workshop\backup
git init --bare backup.git

cd c:\git-workshop\test
git remote add backup "../backup/backup.git"
git push --set-upstream backup main
```

### Worktree

Git worktree kan være nyttig når du er midt i noe og må hjelpe til med noe annet i samme repoet.
Som et alternativ til å stashe endringer / lage en dummy commit kan du lage en ny kopi av alt arbeidet på et eget område.

```shell
git worktree add ..\worktree\test-hotfix
git worktree list
```

Nå kan du jobbe videre i test-hotfix uten at det påvirker arbeidet som er påbegynt i hovedtreet.

```shell
git worktree switch <name>
git worktree remove <name>
```

### Extra?

Forsøk å sette opp signed commits.


```shell
git commit -m "<message>" -S
```

Forsøk å sette opp en commit-hook

```shell
#!/bin/bash
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".

currentBranch="$(git rev-parse --abbrev-ref HEAD)"

if [[ "$currentBranch" != "master" && "$currentBranch" != "main" ]]; then
  [[ "$0" = "$BASH_SOURCE" ]] && exit 1 || return 1;
fi

echo "You are about to commit to $currentBranch. Are you sure? [yes/No]"

exec < /dev/tty

while read -n 1 -r yn; do
  if [ "$yn" = "" ]; then
      yn='N'
  fi
  case $yn in
      [Yy] ) break;;
      [Nn] ) echo;[[ "$0" = "$BASH_SOURCE" ]] && exit 1 || return 1;;
      * ) echo "Please answer y (yes) or n (no):" && continue;
  esac
done

exec <&-
```

