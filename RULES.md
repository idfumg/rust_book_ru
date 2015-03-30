# Правила

## Правила разметки

* **Фигурные скобки в markdown блоках кода** (например \`\`\`{rust, ignore}) - оставлять как написано в оригинале.
* **Ширина текста** - 80.

## Правила перевода

* **Текст в примерах кода** - переводится. [^0]
* **Сообщения об ошибках компилятора** - не переводятся.
* **Специальные markdown заголовки в док-комментариях к коду Rust** (например `# Examples`) - не переводятся. [^1]
* **Цитаты**
  * **"Декоративные" цитаты** - заменять аналогичными по смыслу русскоязычными (при их наличии).
  * **"Cмысловые" цитаты** (несут в себе некий смысл, на который опирается дальнейшее повествование) - переводить близко к тексту.

[^0]: Тонкий момент: не стоит переводить имена вроде Graydon и Niko как Григорий и Николай - это отсылка к создателям языка  
[^1]: В [разделе о документации](https://github.com/rust-lang/rust/blob/master/src/doc/trpl/documentation.md#special-sections) сказано, что они не являются частью синтаксиса, но являются соглашением.

## Правила синтаксиса

* Буква "ё" используется везде, где этого требуют правила русского языка.

## Словарь терминов

crate - крейт  
destructure - деконструкция  
match - сопоставление с образцом  
module - модуль  
slice - срез  

**Если у вас есть вопросы или предложения по правилам - не стесняйтесь, обязательно предлагайте лучшие на ваш взгляд варианты!**