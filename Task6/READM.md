Приложение для блога
Откройте проект, созданный в практической работе 5.

Формы
HTML-формы – становой хребет интерактивных веб-сайтов. Это может быть единственное поле для ввода поискового запроса, как на сайте Google, вездесущая форма для добавления комментария в блог или сложный специализированный интерфейс ввода данных. 
В этой практической работе мы продолжим работу над нашим приложением блога, добавив формы, чтобы пользователь мог создавать, редактировать или удалять любую из записей блога.
Формы очень распространены и очень сложны для правильной реализации. Каждый раз, когда вы принимаете пользовательский ввод, возникают проблемы безопасности (XSS Атаки), требуется правильная обработка ошибок, а также вопросы пользовательского интерфейса о том, как предупредить пользователя о проблемах с формой. Не говоря уже о необходимости перенаправления при успехе.
К счастью для нас, встроенные формы Django абстрагируют большую часть сложности и предоставляют богатый набор инструментов для обработки распространенных случаев использования, работающих с формами.
Чтобы начать, обновите наш шаблон base.html, чтобы отобразить ссылку на страницу для ввода новых сообщений в блоге.
 
post_new — это имя нашего URL, сейчас мы его создадим. 
Добавим новый url для post_new:
 
Ваш url будет начинаться с post / new/, представление называется Blog Create View, и url будет называться post_new. То есть при переходе на ссылку post/new/ будет вызываться встроенный метод as_view у класса BlogCreateView (этот класс мы сейчас создадим).
Теперь давайте создадим наше представление, импортировав новый универсальный класс под названием Create View, а затем создадим его подкласс для создания нового представления с именем BlogCreateView. Напоминаю, что представления создаются в файле views.py.
Сначала подключим класс CreateView:
 
В файле views.py уже есть два представления, добавьте после них третье с названием: BlogCreateView и заполните следующим кодом:
 
В BlogCreateView мы указываем нашу модель базы данных Post, имя нашего шаблона post_new.html и все поля с '  all  ', так как у нас только два: title и author (также можно задать вручную только необходимые поля). 
Последний шаг - создать наш шаблон, который мы назовем post_new.html.
Создайте файл post_new.html в папке templates и заполните его следующим кодом:
 
Давайте разберемся с тем, что мы сделали:
-	В верхней строке мы наследуем от нашего базового шаблона.
-	Используются HTML-теги <form> с методом POST, так как мы отправляем данные. 
GET — метод для чтения данных с сайта. Например, для доступа к указанной странице. Он говорит серверу, что клиент хочет прочитать указанный документ. На практике этот метод используется чаще всего, например, в интернет-магазинах на странице каталога. Фильтры, которые выбирает пользователь, передаются через метод GET.
POST — метод для отправки данных на сайт. Чаще всего с помощью метода POST передаются формы.
-	Добавлен {% csrf_token %}, который предоставляется Django для защиты нашей формы от атак межсайтового скриптинга. Вам следует использовать его для всех ваших форм в Django.
-	Затем для вывода данных формы используется {{ form.as_p }}, который отображает форму как серию <p> тегов, каждый <p> из которых содержит одно поле. Просмотреть другие варианты API форм можно здесь: https://django.fun/ru/docs/django/4.1/ref/forms/api/ или здесь: https://djangodoc.ru/3.2/ref/forms/api/
-	Наконец, указываем тип ввода submit и присваиваем ему значение "Save" (тип submin определяет кнопку для предоставление данных формы в обработчик форм).
Можете запустить сервер и проверить работу нового окна.
 
Попробуйте сохранить запись и у вас появится ошибка:
 
Что же произошло?
Сообщение об ошибке Django является весьма полезным. Он жалуется, что мы не указали, куда отправить пользователя после успешной отправки формы. Давайте отправим пользователя на страницу сведений после успешной отправки; таким образом они смогут увидеть свою завершенную публикацию.
Мы можем следовать предложению Django и добавить get_absolute_url в нашу модель. Это лучшая практика, которую вы всегда должны соблюдать. Он устанавливает канонический URL- адрес для объекта, поэтому, даже если структура ваших URL-адресов изменится в будущем, ссылка на конкретный объект будет одинаковой. Короче говоря, вы должны добавить get_absolute_url() и str_ _ () метод к каждой модели, которую вы пишете.
Откройте models.py файл. Добавьте импорт во второй строке для reverse:
 
Также добавьте новый метод get_-absolute_url:
 
Reverse - очень удобная утилита Django, позволяющая ссылаться на объект по имени шаблона URL, в данном случае post_detail. Если вы вспомните наш шаблон URL, то он выглядит следующим образом:
 
Это означает, что для того, чтобы этот маршрут работал, мы должны также передать аргумент с ключом pk или первичным ключом объекта. Поэтому мы сообщаем Django, что конечным местоположением записи Post является ее представление post_detail, которое представляет собой posts / <int: pk> /, поэтому маршрут для первой созданной нами записи будет находиться в posts / 1.
Попробуйте создать новую запись в блоге еще раз и в случае успеха вы будете перенаправлены на страницу подробного просмотра, где размещена запись.
 
Вы также заметите, что наш предыдущий пост в блоге тоже есть. Он был успешно отправлен в базу данных, но Django не знал, как перенаправить нас после этого.
 
Хотя мы можем обратиться к администратору Django, чтобы удалить нежелательные сообщения, но лучше добавить формы, чтобы пользователь мог обновлять и удалять существующие сообщения непосредственно с сайта.

Форма обновления
Процесс создания формы обновления, чтобы пользователи могли редактировать сообщения в блоге, должен быть знакомым. Мы снова будем использовать встроенное общее представление на основе классов Django UpdateView и создадим необходимый шаблон, URL и представление.
Для начала добавьте новую ссылку на post_detail.html, чтобы возможность редактирования сообщения блога появилась на отдельной странице блога.
 
Мы добавили ссылку, используя <a href> ... </a> и тег движка шаблонов Django 
{% url ...%}. В нем мы указали целевое имя нашего URL, которое будет называться post_edit, а также передали необходимый параметр, который является первичным ключом поста post.pk.
Теперь нужно создать шаблон для нашей страницы редактирования с именем post_edit.html. Добавьте новый файл post_edit.html папку templstes и напишите в файле следующий код:
 
Мы снова используем теги HTML <form></form>, csrf_token от Django для безопасности, form.as_p, чтобы отобразить наши поля формы с тегами абзаца, и, наконец, дать ему значение “Update” на кнопке отправки.
Теперь к нашему представлению (файл view.py) нужно импортировать UpdateView на второй строке с верху, а затем создать его подкласс в нашем новом представлении BlogUpdateView.
 
 
Обратите внимание, что в представлении BlogUpdateView мы явно перечисляем поля, которые мы хотим использовать ['title', 'body'], а не "__all__". Это потому, что мы предполагаем, что автор сообщения не меняется; мы хотим, чтобы только заголовок и текст были редактируемыми.
Заключительный этап — это обновление вашего файла urls.py:
 
Мы создали новый шаблон url для /post/pk / edit и дали ему имя post_edit.
Теперь, если вы нажмете на запись в блоге, вы увидите нашу новую кнопку редактирования.
 
Если вы нажмете на "+ Edit Blog Pos", вы будете перенаправлены на http://127.0.0.1:8000/post/1/ edit/if это ваш первый пост в блоге.
 

Обратите внимание, что форма предварительно заполняется существующими данными нашей базы данных для публикации. Давайте внесем изменения и после нажатия кнопки "Update" мы перенаправляемся в подробный вид поста, где вы можете увидеть изменения. Это из-за нашей установки get_absolute_url. Перейдите на главную страницу, и вы увидите изменения рядом со всеми остальными записями.

Форма удаления
Процесс создания формы для удаления записей блога очень похож на процесс обновления записи. Мы будем использовать еще одно общее представление на основе классов, DeleteView, для создания представления, url-адреса и шаблона функциональности.
Для начала добавьте ссылку для удаления записей блога на вашей странице блога, разместите ее под ссылкой для редактирования.
 
Затем создайте новый файл для нашего шаблона удаления страницы. Назовите его post_delete.html и заполните следующим кодом:
 
Обратите внимание, что мы используем post.title для отображения заголовка нашего блога. Мы также могли бы просто использовать object.title, поскольку он также предоставляется DetailView.
Теперь обновите наш файл views.py, импортировав DeleteView и reverse_lazy вверху, затем создайте новое представление, которое будет подклассом DeleteView, назовите его BlogDeleteView.
 
 
Мы используем reverse_lazy вместо просто reverse, чтобы он не выполнял перенаправление URL, пока наше представление не завершило удаление сообщения в блоге.
Наконец, добавьте URL для нового шаблона, так же как мы добавляли url для редактирования. URL должен выглядеть следующим образом:
  
Вызов метода и атрибут name оформите самостоятельно.
Запустите сервер и проверьте что на сообщении в блоге есть ссылка для удаления сообщения. Проверьте работоспособность удаления.
Отправьте работу на GitHub.
