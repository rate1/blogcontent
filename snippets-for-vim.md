Title: Сниппеты для Vim с помощью плагина snipmate (настройка для Pelican и Assembler)
Summary: Установка плагина snipmate для vim и настройка своих сниппетов для Pelican и Assembler.
Date: 2020-12-08
Author: rate1  
Category: Vim
Tags: vim, pelican, assembler, masm32
Slug: snippets-for-vim
Status: published
	
## Сниппеты
**Сниппеты** - это куски кода которые приходится часто использовать и поэтому целесообразно обзавестись инструментом, для их автоматической подстановки в текст.  
С помощью сниппетов реализуем автоматическую подставновку шапки для markdown-статей блога на Pelican и вставку каркаса asm-файла.  
### Плагин snipmate для vim
В редакторе vim для этой цели можно использовать плагин [snipmate](https://github.com/garbas/vim-snipmate "Страница плагина snipmate"). Для корректной работы этого плагина потребуется установить два дополнительный плагина и еще один плагин с библиотекой готовых сниппетов под разные языки программирования. Добавляем в файл .vimrc установку плагинов:  
```
:::vim

call plug#begin('~/.vim/plugged')
" Plug for snippets
Plug 'MarcWeber/vim-addon-mw-utils'
Plug 'tomtom/tlib_vim'
Plug 'garbas/vim-snipmate'
Plug 'honza/vim-snippets'
call plug#end()
```
После этого вводим команду ```:PlugInstall``` и наблюдаем, как скачиваются нужные плагины. Теперь в папке **~/.vim/plugged/vim-snippets/snippets** содержатся файлы настройки сниппетов под разные языки программирования. Эти файлы имеют расширение .snippets, а в названии содержат название языка программирования:  

![Файлы .snippets в папке snippets](/images/snippets-files.jpg "Файлы .snippets")

В эти файлы и будем прописывать нужные нам сниппеты. Для начала подготовим сниппет для файлов markdown.  
#### Настройка сниппета для markdown-файлов  
Откроем файл markdown.snippets, здесь Вы увидите множество уже готовых сниппетов, изучив которые значительно облегчите себе жизнь при работе с файлами расширения .md. Спустимся в самый конец и добавим свой сниппет, который будет создавать шапку статьи для блога, генерируемого с помощью Pelican:  
```
# Pelican snippet
snippet pelican
		Title: ${1:title}
		Summary: ${2:description}
		Date: `strftime("%Y-%m-%d")`
		Author: rate1  
		Category: ${3}
		Tags: ${4}
		Slug: ${5:slug-for-file}
		Status: draft
		
		${6}
```  
Теперь, если при работе с markdown-файлом в режиме вставки можно написать слово **pelican** и нажать на клавишу <Tab>, это приведет к автоматической подстановке нашего сниппета в файл. Курсор будет в положении, обозначенном в нашем сниппете меткой ${1:title}. После заполнения первой метки нажмите повторно на клавишу <Tab> и это переместит активное поле ввода к следующей метке. Обратите внимание, что конструкция `strftime("%Y-%m-%d")` позволяет автоматически подставлять текущую дату.  
#### Настройка сниппетов для asm-файлов  
В списке snippet-файлов отсутствует файл для языка программирования Assembler. Мы это исправим. Создадим файл **asm.snippets** следующего содержания:  
```
# skelet
snippet skelet
		.586P
		.model flat,stdcall
		;------------------
		_DATA SEGMENT
		
		_DATA ENDS

		_TEXT SEGMENT
		START:
		${0}
		_TEXT ENDS
		END START
```  
Это позволит вставлять в .asm файл указанную структуру (в данном примере скелет файла для ассемблера masm32) вводом слова **skelet** и нажатием на клавишу Tab.  
Следующий сниппет позволит вставлять шаблон функции:  
```
# proc
snippet proc
		${1:name} proc
			push ebp
			mov ebp, esp
			push ebx
			push esi
			push edi
			;--------------------------
			${3:code}
			;--------------------------
			pop edi
			pop esi
			pop ebx
			mov esp, ebp
			pop ebp
			ret ${2:number_of_parameters*4}
		$1 endp
```
