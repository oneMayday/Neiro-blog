<h1>ALT stories</h1>
<h2>Описание:</h2>
Проект новостного сайта, контент для которого автоматически генерируются нейросетью Chat GPT-3.
<br></br>
В проекте имеется регистрация пользователей, сброс пароля, подписка на рассылку, генерация контента путём обращения к API Chat GPT-3. Обновление происходит 1 раз в сутки, после обновления и публикации статей (через админ-панель) происходит рассылка о выходе нового контента подписавшимся пользователям.
<br></br>
Задачи генерации контента и рассылки реализованы через очередь Celery, расписание задач - Celery-beat, в качестве брокера сообщений используется Redis. 
<br></br>
<p align="center"> 
<img src="https://i.postimg.cc/ht2sy2K1/main.png" width='32%' height='32%'>
<img src="https://i.postimg.cc/zv9GBGGd/cats.png" width='32%' height='32%'>
<img src="https://i.postimg.cc/Qxfr0t3v/posts.png" width='32%' height='32%'>
</p>

<h2>Порядок установки:</h2>
<details>
<summary>Установка виртуального окружения и зависимостей.</summary>

	
Клонируем репозиторий:
	
	
	https://github.com/oneMayday/AI-newspaper.git
	
Создаем виртуальное окружение и активируем его:
	
	
	python -m venv venv
	Windows: venv\Scripts\activate.bat
	Linux и MacOS: source venv/bin/activate

Переходим в директорию проекта и устанавливаем зависимости:


	pip install -r requirements.txt
	
Переходим в директорию newspaper.


	Файл example.env переименовываем в .env, прописываем в нём свои ключи и данные SMTP сервера.
	
Выполняет миграции:


	python manage.py migrate
	
Запускаем сервер:


	python manage.py runserver

</details>


<details>
<summary>Установка и настройка Redis, Celery и Celery-beat.</summary>
<br>
Для работы отложенных задач и задач по расписанию необходимо запустить 3 отдельных сервера,
именно в том порядке, какой указан в инструкции (redis, celery-beat, celery):

Redis:\
Перейти в папку с установленным Redis и последовательно ввести в консоли:

	.\redis-server start
	.\redis-cli
В консоли должен появиться адрес (по умолчанию 127.0.0.1:6379). 
Проверить работу можно командой PING -> сервер должен ответить PONG.

Celery-beat:\
Перейти в папку с проектом (туда, где находится manage.py) и ввести в консоль:

	celery --app newspaper beat -l info

Celery:\
Перейти в папку с проектом (туда, где находится manage.py) и ввести в консоль:

	celery -A newspaper worker --loglevel=info

ВАЖНО! Для использования под Windows нужно импользовать другую команду:

	celery --app=newspaper worker --pool=solo --loglevel=info
</details>
<h2>Рекомендации по настройке:</h2>
<details>
<summary>Настройка запросов для генерации постов.</summary>
<br>
Для получения более оригинальных постов можно усложнить текст запросов к Chat GPT-3.<br><br>
Логика запросов находится в файле ai_posts/chatgpt_services и представлена двумя функциями - chatgpt_get_post_header и chatgpt_get_post_text, для получения заголовка статьи и текста соответственно.
Новый запрос можно прописать в поле 'prompt', чем сложнее будет запрос - тем оригинальнее будет ответ от платформы. О назначении остальных полей можно подробно прочитать на официальном сайте OPENAI.
</details>

<details>
<summary>Настройка автоматической публикации статей.</summary>
<br>
По умолчанию статьи добавляются в базу данных неопубликованными.\
Добаление в основную ленту осуществляется установкой флага is_published = True в бд или через админку.
Если хочется изменить это поведение (чтобы статьи сразу добавлялись как опубликованные), нужно в ai_posts/tasks изменить значение в функции:

	@shared_task(name='update_news')
	def update_news():
		...
		new_post = Post(...
			is_published=True
		)
		...
</details>

<details>
<summary>Настройка отложенных задач.</summary>
<br>
Логика задач по расписанию (обновление и рассылка) расположена в файле newspaper/celery.\
Для изменения времени, поменяйте значения в полях 'shedule' обоих функций. Подробнее можно почитать в документации к celery-beat.
</details>
<h2>PS:</h2>
Буду рад любым комментариям и критике :)
