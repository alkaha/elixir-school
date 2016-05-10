---
layout: page
title: Pengendalian Ralat
category: advanced
order: 2
lang: my
---

Walaupun kelazimannya ialah dengan memulangkan tuple `{:error, reason}`, Elixir menyokong kaedah pengecualian dan di dalam pelajaran ini kita akan melihat bagaimana untuk mengendalikan ralat dan juga beberapa mekanisma yang disediakan untuk kita.

Secara am tatacara terpakai di dalam Elixir ialah untuk membuat satu fungsi (`example/1`) yang memulangkana `{:ok, result}` dan `{:error, reason}` dan lagi satu fungsi (`example!/1`) yang memulangkan `result` dalam bentuk mentah atau menimbulkan satu ralat.

Pelajaran ini akan memfokuskan interaksi dengan yang kemudian.

## Isi Kandungan

- [Pengendalian Ralat](#pengendalian-ralat)
- [After](#after)
- [Ralat Baru](#ralat-baru)
- [Throw](#throw)
- [Exit](#Exit)

## Pengendalian Ralat

Sebelum kita boleh mengendalikan ralat kita perlu membuatnya terlebih dahulu dan cara paling mudah ialah menggunakan `raise/1`:

```elixir
iex> raise "Oh no!"
** (RuntimeError) Oh no!
```

Jika kita mahu menetapkan jenis dan mesej untuk satu ralat, kita perlu gunakan `raise/2`:

```elixir
iex> raise ArgumentError, message: "the argument value is invalid"
** (ArgumentError) the argument value is invalid
```

Jika kita tahu ralat mungkin berlaku, kita kendalikannya menggunakan `try/rescue` dan pemadanan corak:

```elixir
iex> try do
...>   raise "Oh no!"
...> rescue
...>   e in RuntimeError -> IO.puts("An error occurred: " <> e.message)
...> end
An error occurred: Oh no!
:ok
```

Kita juga boleh memadankan beberapa ralat di dalam satu 'rescue':

```elixir
try do
  opts
  |> Keyword.fetch!(:source_file)
  |> File.read!
rescue
  e in KeyError -> IO.puts "missing :source_file option"
  e in File.Error -> IO.puts "unable to read source file"
end
```

## After

Kadang-kadang ianya perlu untuk menjalankan beberapa tindakan selepas `try/rescue` tanpa menghiraukan ralat apa yang ditimbulkan.  Untuk ini kita ada `try/after`.  Jika anda biasa dengan Ruby ini adalah lebih kurang sama dengan `begin/rescue/ensure` atau di dalam Java `try/catch/finally`:

```elixir
iex> try do
...>   raise "Oh no!"
...> rescue
...>   e in RuntimeError -> IO.puts("An error occurred: " <> e.message)
...> after
...>   IO.puts "The end!"
...> end
An error occurred: Oh no!
The end!
:ok
```

Ini selalunya digunakan untuk mengendalikan fail atau hubungan yang sepatutnya ditutup:

```elixir
{:ok, file} = File.open "example.json"
try do
   # Do hazardous work
after
   File.close(file)
end
```

## Ralat Baru

Walaupun Elixir mengandungi beberapa jenis ralat yang disiap-pasang seperti `RuntimeError`, kita diupayakan untuk membuat jenis ralat kita sendiri jika sesuatu yang khusus diperlukan.  Membuat ralat baru adalah mudah dengan makro `defexception/1` yang menerima pilihan `:message` untuk menetapkan mesej lalai ralat:

```elixir
defmodule ExampleError do
  defexception message: "an example error has occurred"
end
```

Mari kita cuba gunakan ralat baru ini:

```elixir
iex> try do
...>   raise ExampleError
...> rescue
...>   e in ExampleError -> e
...> end
%ExampleError{message: "an example error has occurred"}
```

## Throw

Lagi satu mekanisma untuk mengendalikan ralat di dalam Elixir ialah `throw` dan `catch`.  Ianya amat jarang digunakan di dalam kod Elixir yang terkini tetapi masih penting untuk diketahui dan difahami.

Fungsi `throw/1` mengupayakan kita untuk menghentikan satu pelakuan dengan satu nilai khas yang boleh di-`catch` dan digunakan:

```elixir
iex> try do
...>   for x <- 0..10 do
...>     if x == 5, do: throw(x)
...>     IO.puts(x)
...>   end
...> catch
...>   x -> "Caught: #{x}"
...> end
0
1
2
3
4
"Caught: 5"
```

Sebagaimana yang disebut, `throw/catch` adalah agak jarang digunakan dan selalunya wujud sebagai sokongan sementara apabila pustaka yang sepatutnya tidak membekalkan API yang cukup.

## Exit

Mekanisma terakhir yang dibekalkan Elixir ialah `exit`.  Signal exit berlaku apabila satu proses mati dan ianya komponen penting di dalam kedayatahanan Elixir.

Untuk 'exit' secara jelas kita boleh gunakan `exit/1`:

```elixir
iex> spawn_link fn -> exit("oh no") end
** (EXIT from #PID<0.101.0>) "oh no"
```

Walaupun kta boleh `try/catch` satu 'exit', ianya _teramat_ jarang sekali digunakan.  Boleh dikatakan dalam semua keadaan ianya lebih berfaedah jika membiarkan supervisor untuk mengendalikan `exit` sesatu proses:

```elixir
iex> try do
...>   exit "oh no!"
...> catch
...>   :exit, _ -> "exit blocked"
...> end
"exit blocked"
```
