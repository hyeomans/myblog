title: 'Elixir in Vagrant: Httpoison, memory and ssl_verify_hostname'
date: 2016-05-30 10:19:04
tags:
---

If you tried to install `httpoison` and when you run `mix deps.compile` or `mix` you get:

```
** (Mix) Could not compile dependency idna, /home/vagrant/.mix/rebar command failed. If you want to recompile this dependency, please run: mix deps.compile idna
```

You can fix this issue by increasing the available memory on your Vagrant file to 1GB:

```
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
```

Run `vagrant reload` in your command line after making this change. Then in your root folder do:

- `rm -rf deps _build`
- `mix deps.get`
- `mix`

If you get another error saying:

```
==> idna (compile)
==> ssl_verify_hostname (compile)
Compiling src/ssl_verify_hostname.erl failed:
...
```

You will need to install `erlang-dev` apt package by running `sudo apt-get install erlang-dev`. The rinse and repeat previous steps on your root folder:

- `rm -rf deps _build`
- `mix deps.get`
- `mix`

You can find my whole setup here [Elixir on Vagrant](http://hyeomans.com/2015/11/10/Elixir-in-Vagrant/)
