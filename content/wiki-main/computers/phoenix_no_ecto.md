+++
date = "2017-01-04T00:52:25-05:00"
title = "Separate Ecto From Phoenix In Umbrella App"
draft = false
tags = ["phoenix", "elixir", "tutorial"]
wikis = ["computers"]
+++

First, create umbrella application:

    $ mix new --umbrella my_app
    $ cd my_app/apps

Next create Phoenix app, excluding Ecto:

    $ mix phoenix.new web --no-ecto

Now we create a new mix project for Ecto, with supervision tree:

    $ mix new db --sup

Finally we need to set up some configuration files.

## Setting Up Db Project

    $ cd db

First in the `db` project (following the
[getting started](https://hexdocs.pm/ecto/getting-started.html) docs for Ecto:

**`mix.exs`** (note that we do not need to register `:ecto` application)

      def application do
        [applications: [:logger, :postgrex],
         mod: {Db, []}]
      end

      defp deps do
        [{:postgrex, "~> 0.13.0"},
         {:ecto, "~> 2.1.1"}]
      end

Run configuration by running:

    $ mix ecto.gen.repo -r Db.Repo

This will populate `lib/db/repo.ex` and  `config.config.exs`.

**`lib/db.ex`**

    children = [
      supervisor(Db.Repo, [])
    ]

**optional**

We probably want configs for multiple environments.

**`config/config.exs`**

Uncomment the last line, or replace the contents of the file with:

    use Mix.Config

    config :db,
      ecto_repos: [Db.Repo]

    # Import environment specific config. This must remain at the bottom
    # of this file so it overrides the configuration defined above.
    import_config "#{Mix.env}.exs"

This will load environment-specific configs.

**`config/dev.exs`**

    use Mix.Config

    config :db, Db.Repo,
      adapter: Ecto.Adapters.Postgres,
      username: "foo",
      database: "myproject_dev",
      hostname: "localhost",
      pool_size: 10

**`config/test.exs`**

    use Mix.Config

    config :db, Db.Repo,
      adapter: Ecto.Adapters.Postgres,
      username: "foo",
      password: "12345",
      database: "myproject_test",
      hostname: "localhost",
      pool: Ecto.Adapters.SQL.Sandbox

    config :logger,
      backends: [:console],
      compile_time_purge_level: :debug

## Set Up Phoenix App

    $ cd ../web

**`mix.exs`**

      def application do
        [mod: {Web, []},
         applications: [:phoenix_ecto, :db]]
      end

      defp deps do
        [{:phoenix_ecto, "~> 3.2.1"},
         {:db, in_umbrella: true}]
      end

**`config/config.exs`**

    config :web,
      ecto_repos: [Db.Repo]

    # Configure phoenix generators
    config :phoenix, :generators,
      migration: false

## Notes

* From now on, when running `mix phoenix.gen.html` or `mix phoenix.gen.json`,
pass the `--no-model` option.

* Since app is umbrella app now, remember to `cd` into phoenix app directory
before running generators, or files will be generated at umbrella's top level.

## Questions

* Is it there a way to make it so that there's no need for `--no-model` option
every time?

