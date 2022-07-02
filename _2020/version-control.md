---
layout: lecture
title: "Контроль версий (Гит)"
date: 2020-01-22
ready: true
video:
  aspect: 56.25
  id: 2sjqTHE0zok
---

Системы контроля версий (СКВ, version control systems) - это инструменты, 
используемые для отслеживания изменений в исходном коде (или других файлах и 
папках) и управления ими. Как и подразумевает название, СКВ обеспечивают 
хранение истории изменений; более того, они облегчают сотрудничество 
разработчиков.  СКВ записывают изменения в папку, и ее содержимым являются 
серии снимков, где каждый снимок (snapshot, далее мы будем называть их спнепшотами)
включает состояние всех файлов/папок корневой директории. СКВ также хранят 
метаданные, например, кто создал данный снепшот и т.д.

Почему системы контроля полезны? Даже когда Вы работаете самостоятельно, 
они позволяют Вам посмотреть на более поздние снепшоты проекта, вести учет, 
почему те или иные изменения были сделаны, работать сразу в нескольких 
ветках и многое другое. Когда Вы работаете в команде, это бесценный инструмент 
для отслеживания изменений, внесенных другими разработчиками, а также разрешения 
конфликтов, возникающих в процессе параллельной разработки. 

Современные СКВ предоставляют возможность легко (и часто автоматически) ответить 
на такие вопросы, как:
- Кто написал данный модуль?
- Когда эта конкретная строка этого конкретного файла была изменена? Кем? Почему?
- В ходе последних 1000 изменений, когда/почему определенный юнит тест начал падать?

Несмотря на существование других СКВ, **Git** де факто является стандартом и данный 
[XKCD комикс](https://xkcd.com/1597/) отражает его репутацию. 

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

-- Это гит. Он отражает совместную работу над проектом в виде чудесной графовой структуры
дерева.

-- Здорово, как мы это используем?

-- Ни малейшего представления не имею. Просто запомни эти команды и напиши их 
для синхронизации. Если будут ошибки, сохрани свою работу где-то в другом месте, 
удали проект и загрузи новую версию.

Так как интерфейс гита является некоторой абстракцией, изучение его “сверху вниз” 
(начиная с его интерфейса) может привести в замешательство. Можно конечно запомнить
полезные команды и думать о них как о волшебных заклинаниях, следуя примеру из 
комикса каждый раз, как что-то пойдет не так. 

Несмотря на ужасный интерфейс, дизайн и идеи, лежащие в основе гита, прекрасны.
И если интерфейс придется _запомнить_, идею можно _понять_. Для этого мы даем 
объяснение гита “снизу вверх”, начиная с его модели данных и позднее касаясь 
интерфейса командной строки. Когда мы разберемся с моделью данных, Вам будет 
легче понять, как команды взаимодействуют с ней. 

# Модель данных

СКВ предоставляет массу возможностей своим пользователям. Гит обладает хорошо
продуманной моделью, благодаря которой существуют опции работы с ветками (branches),
просмотра истории изменений, ведения разработки в команде.

## Снепшоты

История файлов и каталогов проекта (корневого каталога) в Git представляется
серией снепшотов. В терминологии Git файл называется "blob", который является
просто набором байтов. Каталог называется "tree" (деревом), он сопоставляет имена
файлам и каталогам (таким образом каталоги могут содержать другие каталоги). Снимок -
состояние корневого каталога под управлением системы контроля версий. Рассмотрим
следующий пример:

```
<корень> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, состоит из = "hello world")
|
+- baz.txt (blob, состоит из = "git is wonderful")
```

Корневой каталог содержит два элемета: каталог "foo" (который в свою очередь
состоит из файла "bar.txt") и файл "baz.txt".

## Структура истории изменений; связь снепшотов

Каким образом СКВ должна связывать снимки? Примером простого подхода может
служить линейная упорядоченность снимков в хронологическом порядке. По многим
причинам Гит не использует подобный подход.

В Гите история представляется ориентированным ациклическим графом снимков. Возможно
звучит заумно, но не надо пугаться. Это лишь значит, что каждый снимок в гите имеет
набор "родителей", снимков, предшествующих рассматриваемому. Это набор родителей,
а не один (как было бы в случае линейной истории), потому что снэпшот может быть
результатом нескольких родителей, как в случае соединения (сливания) двух параллельных
веток разработки.

В гите такие снэпшоты называются коммитами. Представление истории коммитов может
выглядеть следующим образом:

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

В ASCII рисунке выше `o` является некоторым коммитом (снепшотом). Стрелки указывают
на родителей каждого из коммитов (изображено отношение "предшествует", не "следует").
После третьего коммита история делится на две ветки. Такой сценарий возможен, когда,
к примеру, две новые функциональности разрабатываются параллельно, независимо друг
от друга. В будущем эти ветки могут быть объеденины чтобы сделать снимок, который
включает в себя оба новшества, создавая новую историю с merge-коммитом, выделенным
жирным ниже:

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</code>
</pre>

Коммиты в Гите неизменяемы. Это не значит, что ошибки не могут быть исправлены; это
лишь означает, что "изменения" истории коммитов есть создание новых коммитов, и
ссылки (описанные далее) обновляются, указывая на новые коммиты.

## Псевдокод модели данных

Может стать яснее, если взглянуть на модель данных Гита описанных в псевдокоде:

```
// файл - это набор байтов
type blob = array<byte>

// каталог содержит именованные каталоги и файлы
type tree = map<string, tree | blob>

// коммит содержит родителя, метаданные и корневой каталог
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

Так выглядит простая и ясная модель истории.

## Объекты и адресация по содержимому

Объект - это файл, каталог или коммит:

```
type object = blob | tree | commit
```

В хранилище данных Гита, у всех объектов есть адресация по содержимому
с помощью их [SHA-1 хэша](https://en.wikipedia.org/wiki/SHA-1).

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Файлы, каталоги и коммиты унифицированы по принципу: все они являются объектами.
Когда они ссылаются на другие объекты, они не на самом деле _содержат_ их, но
содержат ссылку на них с помощью их хэша. 

Например, каталог в примере [выше](#снепшоты)
(визуализация с помощью `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`),
выглядит следующим образом:

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

Каталог сам по себе содержит указатели на его содержимое, `baz.txt` (файл) и 
`foo` (каталог). Если мы посмотрим на содержимое по адресу хэша, относящегося к
baz.txt с помощью `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`, мы 
получим следующее:

```
git is wonderful
```

## Ссылки 

Итак, все спепшоты могут быть идентифицированы при помощи SHA-1 хэшей. Это неудобно,
ведь люди не слишком-то хороши в запоминании строк из 40 шестнадцатеричных символов.

Гит решил эту проблему созданием человекочитыаемых названий для SHA-1 хэшей, так 
называемых ссылок. Ссылки - это указатели на коммиты. В отличие от неизменяемых 
объектов ссылки изменяемы (могут быть обновлены для указания на новый коммит).
Например, `master`(или `main`) как правило ссылается на последний коммит в главной 
ветке разработки.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

Используя это, Гит может задействовать такие привычные для человека имена,
как "master", для ссылки на определенный коммит в истории вместо длинной
шестнадцатеричной строки.

Есть один нюанс, часто мы хотим узнать "а где я сейчас нахожусь?" в истории, 
чтобы при создании снэпшота мы знали, к чему он будет относиться (что будет в 
поле "родители" у коммита). В Гите это "а где я сейчас нахожусь?" является
специальной ссылкой под названием "HEAD".


## Репозитории

Наконец, мы можем определить что (приблизительно) такое Git _репозиторий_ : 
это объекты и ссылки.

На диске Гит хранит объекты и ссылки - это все, что существует для модели
данных Гита. Все `git` команды относятся к какой-нибудь манипуляции с DAG
(directed acyclic grap, направленный ациклический граф) коммитов путем добавления
нового объекта и добавления/обновления ссылок.

Когда Вы пишете любую команду, подумайте какую манипуляцию она проводит с
графовой структурой данных. И наоборот, если Вы хотите произвести какое-то 
изменение над DAG коммитов, например, "отменить незакоммиченные изменения и
перенести ссылку 'master' на коммит `5d83f9e`", вероятно существует команда, 
чтобы это сделать (в данном случае `git checkout master; git reset
--hard 5d83f9e`).

# Стейджинг

Еще одна концепция, по сути не относящаяся к модели данных, но являющаяся частью
интерфейса для создания коммитов.

Вы можете представлять себе реализацию процесса создания снимков как некоторую
команду "создай снепшот", которая создает новый снепшот на основании _текущего
состояния_ рабочей директории. Некоторые системы контроля версий так и делают, но
не Гит. Мы хотим "чистые" снимки, и не всегда текущее состояние для этого подходит.
Например, вообразите сценарий, где Вы реализовали два разных функционала, и Вы
хотели бы создать для них два разных коммита. Или представьте, что у Вас по всему
коду есть отладка с помощью print вместе с исправленной ошибкой, и Вы хотите 
закоммитить это исправление без всех print'ов. 

Гит приспособлен для таких сценариев: есть возможность включить в следующий коссит 
нужные Вам изменения с помощью так называемого механизма "зона стейджинга".

# Работа с Гитом через командную строку

Чтобы не повторяться, мы не будем объяснять приведенные далее команды в деталях.
Посмотрите лекцию или воспользуйтесь крайне рекомендуемым
[Pro Git](https://git-scm.com/book/en/v2) за подробностями.

## Основы

Команда `git init` создает новый Гит репозиторий с метаданными, сохраненными
в директории `.git`:

```console
$ mkdir myproject
$ cd myproject
$ git init
Initialized empty Git repository in /home/missing-semester/myproject/.git/
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Как мы можем интерпретировать данный вывод? "No commits yet" (пока нет коммитов) 
обозначает, что наша история версий пуста. Давайте это исправим.

```console
$ echo "hello, git" > hello.txt
$ git add hello.txt
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   hello.txt

$ git commit -m 'Initial commit'
[master (root-commit) 4515d17] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
```

И вот, мы добавили файл в зону стейджинга и затем закоммитили это
изменение, добавив коммиту простое сообщение "Initial commit". Если
мы не укажем опцию `-m`, Гит откроет текстовый редактор, чтобы Вы
написали сообщение коммита.

Теперь когда у нас есть непустая история версий, мы можем ее визуализировать.
Визуализация с помошью DAG может быть особенно полезна для понимания текущего
статуса репозитория и связи с моделью данных Гита.

Команда `git log` визуализирует историю. По дефолту это плоская версия, скрывающая
графовую структуру. Если Вы используете команду `git log --all --graph --decorate`,
она покажет полную версию репозитория в графовом представлении.

```console
$ git log --all --graph --decorate
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa (HEAD -> master)
  Author: Missing Semester <missing-semester@mit.edu>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

Сейчас это не выглядит как граф, так как содержит всего один узел. Давайте 
произведем еще какие-нибудь изменения, создадим коммит, и визуализируем историю
еще раз.

```console
$ echo "another line" >> hello.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add hello.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   hello.txt

$ git commit -m 'Add a line'
[master 35f60a8] Add a line
 1 file changed, 1 insertion(+)
```

Now, if we visualize the history again, we'll see some of the graph structure:

```
* commit 35f60a825be0106036dd2fbc7657598eb7b04c67 (HEAD -> master)
| Author: Missing Semester <missing-semester@mit.edu>
| Date:   Tue Jan 21 22:26:20 2020 -0500
|
|     Add a line
|
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa
  Author: Anish Athalye <me@anishathalye.com>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

Also, note that it shows the current HEAD, along with the current branch
(master).

We can look at old versions using the `git checkout` command.

```console
$ git checkout 4515d17  # previous commit hash; yours will be different
Note: checking out '4515d17'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 4515d17 Initial commit
$ cat hello.txt
hello, git
$ git checkout master
Previous HEAD position was 4515d17 Initial commit
Switched to branch 'master'
$ cat hello.txt
hello, git
another line
```

Git can show you how files have evolved (differences, or diffs) using the `git
diff` command:

```console
$ git diff 4515d17 hello.txt
diff --git c/hello.txt w/hello.txt
index 94bab17..f0013b2 100644
--- c/hello.txt
+++ w/hello.txt
@@ -1 +1,2 @@
 hello, git
 +another line
```

{% endcomment %}

- `git help <command>`: get help for a git command
- `git init`: creates a new git repo, with data stored in the `.git` directory
- `git status`: tells you what's going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit
    - Write [good commit messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
    - Even more reasons to write [good commit messages](https://chris.beams.io/posts/git-commit/)!
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate`: visualizes history as a DAG
- `git diff <filename>`: show changes you made relative to the staging area
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD and current branch

## Branching and merging

{% comment %}

Branching allows you to "fork" version history. It can be helpful for working
on independent features or bug fixes in parallel. The `git branch` command can
be used to create new branches; `git checkout -b <branch name>` creates and
branch and checks it out.

Merging is the opposite of branching: it allows you to combine forked version
histories, e.g. merging a feature branch back into master. The `git merge`
command is used for merging.

{% endcomment %}

- `git branch`: shows branches
- `git branch <name>`: creates a branch
- `git checkout -b <name>`: creates a branch and switches to it
    - same as `git branch <name>; git checkout <name>`
- `git merge <revision>`: merges into current branch
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base

## Remotes

- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
- `git fetch`: retrieve objects/references from a remote
- `git pull`: same as `git fetch; git merge`
- `git clone`: download repository from remote

## Undo

- `git commit --amend`: edit a commit's contents/message
- `git reset HEAD <file>`: unstage a file
- `git checkout -- <file>`: discard changes

# Advanced Git

- `git config`: Git is [highly customizable](https://git-scm.com/docs/git-config)
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: [specify](https://git-scm.com/docs/gitignore) intentionally untracked files to ignore

# Miscellaneous

- **GUIs**: there are many [GUI clients](https://git-scm.com/downloads/guis)
out there for Git. We personally don't use them and use the command-line
interface instead.
- **Shell integration**: it's super handy to have a Git status as part of your
shell prompt ([zsh](https://github.com/olivierverdier/zsh-git-prompt),
[bash](https://github.com/magicmonty/bash-git-prompt)). Often included in
frameworks like [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).
- **Editor integration**: similarly to the above, handy integrations with many
features. [fugitive.vim](https://github.com/tpope/vim-fugitive) is the standard
one for Vim.
- **Workflows**: we taught you the data model, plus some basic commands; we
didn't tell you what practices to follow when working on big projects (and
there are [many](https://nvie.com/posts/a-successful-git-branching-model/)
[different](https://www.endoflineblog.com/gitflow-considered-harmful)
[approaches](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)).
- **GitHub**: Git is not GitHub. GitHub has a specific way of contributing code
to other projects, called [pull
requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
- **Other Git providers**: GitHub is not special: there are many Git repository
hosts, like [GitLab](https://about.gitlab.com/) and
[BitBucket](https://bitbucket.org/).

# Resources

- [Pro Git](https://git-scm.com/book/en/v2) is **highly recommended reading**.
Going through Chapters 1--5 should teach you most of what you need to use Git
proficiently, now that you understand the data model. The later chapters have
some interesting, advanced material.
- [Oh Shit, Git!?!](https://ohshitgit.com/) is a short guide on how to recover
from some common Git mistakes.
- [Git for Computer
Scientists](https://eagain.net/articles/git-for-computer-scientists/) is a
short explanation of Git's data model, with less pseudocode and more fancy
diagrams than these lecture notes.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
is a detailed explanation of Git's implementation details beyond just the data
model, for the curious.
- [How to explain git in simple
words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) is a browser-based
game that teaches you Git.

# Exercises

1. If you don't have any past experience with Git, either try reading the first
   couple chapters of [Pro Git](https://git-scm.com/book/en/v2) or go through a
   tutorial like [Learn Git Branching](https://learngitbranching.js.org/). As
   you're working through it, relate Git commands to the data model.
1. Clone the [repository for the
class website](https://github.com/missing-semester/missing-semester).
    1. Explore the version history by visualizing it as a graph.
    1. Who was the last person to modify `README.md`? (Hint: use `git log` with
       an argument).
    1. What was the commit message associated with the last modification to the
       `collections:` line of `_config.yml`? (Hint: use `git blame` and `git
       show`).
1. One common mistake when learning Git is to commit large files that should
   not be managed by Git or adding sensitive information. Try adding a file to
   a repository, making some commits and then deleting that file from history
   (you may want to look at
   [this](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).
1. Clone some repository from GitHub, and modify one of its existing files.
   What happens when you do `git stash`? What do you see when running `git log
   --all --oneline`? Run `git stash pop` to undo what you did with `git stash`.
   In what scenario might this be useful?
1. Like many command line tools, Git provides a configuration file (or dotfile)
   called `~/.gitconfig`. Create an alias in `~/.gitconfig` so that when you
   run `git graph`, you get the output of `git log --all --graph --decorate
   --oneline`. Information about git aliases can be found [here](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias).
1. You can define global ignore patterns in `~/.gitignore_global` after running
   `git config --global core.excludesfile ~/.gitignore_global`. Do this, and
   set up your global gitignore file to ignore OS-specific or editor-specific
   temporary files, like `.DS_Store`.
1. Fork the [repository for the class
   website](https://github.com/missing-semester/missing-semester), find a typo
   or some other improvement you can make, and submit a pull request on GitHub
   (you may want to look at [this](https://github.com/firstcontributions/first-contributions)).
