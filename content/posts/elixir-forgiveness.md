---
title: Handling "no process" errors in Elixir
subtitle: Try asking forgiveness instead of getting permission
date: 2019-12-28T21:04:42+08:00
tags: ["elixir", "erlang"]
---

Let's say we have a PID[^1] from somewhere:

```
iex(1)> pid = :c.pid(0, 124, 0)
#PID<0.124.0>
```

And we want to use it as a GenServer reference:

```
iex(2)> GenServer.call(pid, :do_something)
** (exit) exited in: GenServer.call(#PID<0.124.0>, :do_something, 5000)
    ** (EXIT) no process: the process is not alive or there's no process currently associated with the given name, possibly because its application isn't started
    (elixir) lib/gen_server.ex:1009: GenServer.call/3
```

Oops, our current process crashes. To prevent this, we could check if
the PID actually refers to a process first:

```
iex(3)> Process.alive?(pid) && GenServer.call(pid, :do_something) || :error
:error
```

However, this is a classic [time-of-check to
time-of-use](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use) race
condition because it's possible for the process to die after the
`Process.alive?` check but before the GenServer call, causing it to fail
nevertheless.

## Easier to ask forgiveness than to get permission

To mitigate this possibility, we can follow the EAFP principle, which goes: "It
is easier to ask forgiveness than it is to get permission." 

Instead of first checking if the process is alive (getting permission), we
should directly go ahead and make the GenServer call, but explicitly handle the
failing scenario (asking forgiveness): 

```
iex(4)> try do
...(4)>   GenServer.call(pid, :do_something)
...(4)> catch
...(4)>   :exit, {:noproc, _} -> :error
...(4)> end
:error
```

The two argument version of `catch` is a special form which allows you to catch
values of any kind, including exits
([documentation](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#try/1-catching-values-of-any-kind)).

## Caveats

EAFP is commonly associated with Python, which defines the term in the [glossary
section of its
documentation](https://docs.python.org/3/glossary.html#term-eafp). In the
Erlang/OTP ecosystem (which Elixir is a part of), there's a related philosophy
of  fault tolerance through expecting that errors will happen and being able to
recover from them to a known good state through the [supervison
tree](https://erlang.org/doc/design_principles/sup_princ.html).

In light of that, it might not actually be necessary to handle a process exit
explicitly like the example above---if your process is properly supervised, you
could just [let it crash](https://wiki.c2.com/?LetItCrash). This is especially
true if you don't have any reasonable action to take after handling the error.

[^1]: In these examples above we are using a PID directly, but everything still
  applies even with named processes or via tuples.
