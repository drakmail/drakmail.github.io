---
layout: post
title: Миграция с tumblr на jekyll
tags:
- tumblr
- jekyll
- ruby
---

В связи с недавними новостями о [закрытии Google Reader](http://googleblog.blogspot.ru/2013/03/a-second-spring-of-cleaning.html), решил всё-таки начать отвязываться от облачных серверов. В принципе, так как кроме блога я ничего не размещаю на чужих ресурсах, он и стал сегодняшней утренней жертвой. Да и вообще tumblr явно не совпадает с моим видением блогоплатформы.

Сама по себе миграция очень проста, если не брать в расчет некоторые подводные камни:

```bash
$ gem install jekyll
$ jekyll new blog
$ ruby -rubygems -e 'require "jekyll/migrators/tumblr"; Jekyll::Tumblr.process("http://www.your_blog_url.com")'
```

Но дьявол, как известно, в деталях :)

## Проблемы

**Проблема номер раз:** jekyll говорил при попытке импорта 404. Как оказалось, тут всё просто, имя блога необходимо обязательно указывать без `/` в конце :)

**Проблема номер два:** после указания правильного URL для jekyll, уже ruby ругался на непонятный метод `at`, который применяется к хешу (что-то вроде `undefined method 'at' for #<Hash`). Что это и зачем он там - неясно. Правим исходники (`lib/ruby/gems/2.0.0/gems/jekyll-import-0.1.0.beta1/lib/jekyll/jekyll-import/tumblr.rb`) руками:

```ruby
          if !post["id3-title"].nil?
            title = post["id3-title"]
            content = post.at["audio-player"] + "<br/>" + post["audio-caption"]
          else
            title = post["audio-caption"]
            content = post.at["audio-player"]
          end
```

заменяем на

```ruby
          if !post["id3-title"].nil?
            title = post["id3-title"]
            content = post["audio-player"] + "<br/>" + post["audio-caption"]
          else
            title = post["audio-caption"]
            content = post["audio-player"]
          end
```

**Проблема номер три:** при попытке импорта в markdown выскакивает вот такая ошибка: `in 'gsub!': invalid byte sequence in UTF-8`. Как оказалось, html2text, который нужен для импорта в markdown, не умеет работать с UTF-8 без патчей. Решение - отказаться от импорта в markdown :)

**Проблема номер четыре,** которая хоть и не проблема вовсе, но напрягает: адреса постов после импорта имели страшный вид (`---_--_.html`, примерно такой). Я это поправил быстропатчем:

Шаг 1. Устанавливаем gem russian

```bash
$ gem install russian
```

Шаг 2. Правим код, добавляя транслитерацию заголовка

```ruby
# В начало
require 'russian'

# В район 100-й строки, где генерируется slug
# заменяем slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '') на
slug = Russian.translit(title).downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
```

Собственно, на этом проблемы заканчиваются, и можно спокойно использовать [jekyll](https://github.com/mojombo/jekyll).
