+++
title = "Ecto Callbacks Macro"
draft = false
date = "2016-10-22"
wikis = ["computers"]
tags = ["elixir", "metaprogramming"]
+++

Ecto callbacks (before/after) commit hooks have been deprecated, for a
general good reason. Using callbacks is generally bad and you should never do
it. But sometimes you need to do it because real life.

I wrote this macro this afternoon that implements both atomic and non-atomic
callbacks. Here it is in all its glory. I won't make it a hex package because
it's probably a bad thing to do.

## Usage

In your `YourProject.Model.ex` file, add:

```
defmodule MyProject.Model do
  defmacro __using__(_opts) do
    quote do
      use Ecto.Schema
      import Ecto.Changeset
      use MyProject.Hooks
    end

    @doc """
    Dumb. For piping together methods in an `atomic_after_*` function call.

    ## Example

      first_callback()
      |> hook_glue()
      |> second_callback()
    """
    def hook_glue({:ok, struct}), do: struct
    def hook_glue(struct), do: struct
  end
```

## Implementation

```
defmodule MyProject.Hooks do
  @moduledoc """
  This module macro defines the following functions within the model,
  delegating to `Repo`, but allowing us to add `after_*` hooks,
  which are no longer supported in Ecto
  (http://blog.plataformatec.com.br/2015/12/ecto-v1-1-released-and-ecto-v2-0-plans/)

      delete,
      delete!,
      delete_all,
      insert,
      insert!,
      insert_all,
      insert_or_update,
      insert_or_update!,
      update,
      update!,
      update_all

  ## Hooks

  `after_*` will perform action after transaction is done.
  `atomic_after_*` will perform action within the same transaction, at the end.

  When using `atomic_after`, make sure to return the resulting struct in
  the success case, in its tuple format `{:ok, res}` or `{:error, error}`.

  The return value of the transaction is what will be returned by the action.

  ## Usage

  Will print console message after updating:

  **NOTE:** Remember to cover general case after.

      defmodule MyModel do
        use MyProject.Model # which calls `use MyProject.Overrides`

        defp after_update(result, changeset) do
          IO.puts("updating!")
        end

        defp atomic_after_update(r = {:error, _}, _), do: r
        defp atomic_after_update(res, ch) do
          case OtherModel.get!(1).name do
            "bob" -> Repo.rollback(:cant_update_if_name_is_bob)
            _ -> res
          end
        end

      end

      MyModel.update!(ch)
      # -> "updating!"
      %MyModel{...}
  """

  defmacro __using__(_) do
    actions = [:delete,
               :delete!,
               :delete_all,
               :insert,
               :insert!,
               :insert_all,
               :insert_or_update,
               :insert_or_update!,
               :update,
               :update!,
               :update_all]

    # For the love of God, do not touch until this is tested!
    quote do
      alias MyProject.Repo

      unquote do
        Enum.map(actions, fn action ->
          quote do
            def unquote(action)(changeset, opts \\ []) do
              fwd = fn ->
                res = Repo.unquote(action)(changeset, opts)
                after_res = unquote(:"atomic_after_#{action}")(res, changeset)

                # Handle whether the action is with bang for or not
                unquote do
                  if String.last(Atom.to_string(action)) == "!" do
                    quote do
                      transaction_result =
                        case after_res do
                          {:ok, res} -> res
                          {:error, error} -> raise error
                        end
                    end
                  else
                    quote do
                      transaction_result = after_res
                    end
                  end
                end
                transaction_result
              end

              result =
                case Repo.transaction(fwd) do
                  # Covers e.g insert / update
                  {:ok, {:ok, res}} -> {:ok, res}
                  # Just in case
                  {:ok, {:error, error}} -> {:error, error}
                  # Covers insert! update! which return struct
                  {:ok, res} -> res
                  # Covers rollback which returns error
                  {:error, error} -> {:error, error}
                end

              unquote(:"after_#{action}")(result, changeset)
              result
            end
          end
        end)
      end

      unquote do
        Enum.map(actions, fn action ->
          quote do
            defp unquote(:"after_#{action}")(_res, _original_struct), do: nil
          end
        end)
      end

      unquote do
        Enum.map(actions, fn action ->
          if String.last(Atom.to_string(action)) == "!" do
            quote do
              defp unquote(:"atomic_after_#{action}")(res, _) do
                {:ok, res}
              end
            end
          else
            quote do
              defp unquote(:"atomic_after_#{action}")(res, _), do: res
            end
          end
        end)
      end

      defoverridable [after_delete: 2,
                      atomic_after_delete: 2,
                      after_delete!: 2,
                      atomic_after_delete!: 2,
                      after_delete_all: 2,
                      atomic_after_delete_all: 2,
                      after_insert: 2,
                      atomic_after_insert: 2,
                      after_insert!: 2,
                      atomic_after_insert!: 2,
                      after_insert_all: 2,
                      atomic_after_insert_all: 2,
                      after_insert_or_update: 2,
                      atomic_after_insert_or_update: 2,
                      after_insert_or_update!: 2,
                      atomic_after_insert_or_update!: 2,
                      after_update: 2,
                      atomic_after_update: 2,
                      after_update!: 2,
                      atomic_after_update!: 2,
                      after_update_all: 2,
                      atomic_after_update_all: 2]
    end
  end
end
```
