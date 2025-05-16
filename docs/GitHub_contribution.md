# **Внесение вклада в открытый репозиторий**

1. В качестве репозитория, в который я хочу сделать вклад я выбрал **homebrew-core**

![Вид репозитория](/Users/zkhr/Documents/Education/Практика/Images/HomebrewOverview.png)

2. Затем я сделал fork репозитория

![Вид окна fork](/Users/zkhr/Documents/Education/Практика/Images/Forking.png)

3. ***После, я клонировал репозиторий локально командой:*** **git clone https://github.com/le-zkhr/homebrew-core.git**

4. Дождавшись клонирования, я решил внести изменения в файл по пути ***Formula/r/rbenv-aliases.rb***

5. Я добавил в test do ещё одну проверку: выполнение команды rbenv alias, которая должна существовать после установки. Она выводит справку по использованию (что также подтверждает, что плагин установлен и подключён правильно).

***Код до:***
```Ruby
class RbenvAliases < Formula
  desc "Make aliases for Ruby versions"
  homepage "https://github.com/tpope/rbenv-aliases"
  url "https://github.com/tpope/rbenv-aliases/archive/refs/tags/v1.1.0.tar.gz"
  sha256 "12e89bc4499e85d8babac2b02bc8b66ceb0aa3f8047b26728a3eca8a6030273d"
  license "MIT"
  revision 1
  head "https://github.com/tpope/rbenv-aliases.git", branch: "master"

  bottle do
    rebuild 1
    sha256 cellar: :any_skip_relocation, all: "1478142388cffd4c60833cdc2b6e7f3bcac3f6e8b15e095167718ceb0cd7c237"
  end

  depends_on "rbenv"

  def install
    prefix.install Dir["*"]
  end

  test do
    assert_match "autoalias.bash", shell_output("rbenv hooks install")
  end
end
```

***Код после:***
```Ruby
class RbenvAliases < Formula
  desc "Make aliases for Ruby versions"
  homepage "https://github.com/tpope/rbenv-aliases"
  url "https://github.com/tpope/rbenv-aliases/archive/refs/tags/v1.1.0.tar.gz"
  sha256 "12e89bc4499e85d8babac2b02bc8b66ceb0aa3f8047b26728a3eca8a6030273d"
  license "MIT"
  revision 1
  head "https://github.com/tpope/rbenv-aliases.git", branch: "master"

  bottle do
    rebuild 1
    sha256 cellar: :any_skip_relocation, all: "1478142388cffd4c60833cdc2b6e7f3bcac3f6e8b15e095167718ceb0cd7c237"
  end

  depends_on "rbenv"

  def install
    prefix.install Dir["*"]
  end

  test do
    # Проверка, что хук autoalias присутствует
    assert_match "autoalias.bash", shell_output("rbenv hooks install")

    # Проверка, что команда rbenv alias работает (выдаст usage или help)
    output = shell_output("rbenv alias 2>&1", 1)
    assert_match "Usage", output
  end
end
```

6. После, я сохранил файл, связал локальный репозиторий с удаленным и отправил изменения на сервер:
```Bash
git add .
git commit -m "Checking for usage help output"
git push origin master
```

7. После отправки изменений я перешел в свой fork, нажал на кнопку ***Compare & pull request***, описал свои изменения и нажал ***Compare & pull request***.

8. Теперь авторы репозитория могут просмотреть мои изменения, попросить меня что-либо исправить, либо принять pull request сразу и тогда мои изменения внесутся в удаленный репозиторий.