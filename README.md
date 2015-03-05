#vsegda REST api v1.12
##(2015-03-05 11:16:00)

На сайте http://api.taxi21.ru поднят REST API для взаимодействия с __Информационной Системой Такси (ИСТ)__

В следующем разделе описание доступных ресурсов.

---------

###Данные в каждом заголовке HTTP запроса
    Content-Type: application/json
	Content-Language: язык возвращаемых данных // не работает
    Cookie: sessionid=###### // id сессии
    Apikey: ######################## // key

### Регистрация/вход классическая схема

####Поиск пользователя

    URL: /v1/Users/
    
    method GET
    GET params:
        lgn: "телефон 10 знаков" // 

    success: {
        // данные пользователя
        "lgn": "--",
        "tme_reg": "", // дата/время регистрации
        "tme_upd": "" // дата/время последнего обновления
    }

####Вход, изменение учётных данных
    
    URL: /v1/User/
    
    method POST: { // - вход при помощи логина/пароля
        "lgn": "9128649400",
        "pwd": "5ghJgs3"
    }
    
    success: {
        ... // данные пользователя
    }
    
    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

    method PUT: { // изменение учётных данных
        "lgn": "9128649400",
        "pwd": "5ghJgs3"
        "email": "user@mail.com",
        "birthday": "1980-02-29",
        "gender": "женск."
        "name": {
            "value": "Полное имя",
            "first": "Имя",
            "middle": "Отчество",
            "last": "Фамилия"
        },
        "adds": [
            {
                "value": "наименование",
                "twn_id": "id города",
                "stt": "Улица",
                "hse": "Дом",
                "ent": "Куда подъехать"
            },
            ...
        ]
    }
    
    success: {
        // расширенная информация о пользователе
        // привязанные к телефонам карты, рейтинги, остаток средств и т.д.
    }

    error: {
        "code": #,
        "detail": "Сообщение об ошибке"
    }

    method GET - проверка, выполнен ли вход, если да, получение учётных данных

    success: {
        "lgn": "9128649400",
        "email": "user@mail.com",
        "birthday": "1980-02-29",
        "gender": "женск."
        "tels": [ // привязанные номера телефонов
        	{
        		"tel": "---",
        		"srv_id": ---, // приоритетная служба
        		"rtg": ---, // рейтинг на данной службе
        		"cards": [ // привязанные карты
        			{
        				"num": "---",
        				"type": --,
        				"rem": "---",
        				"srv_id": ---,
        				"sum": --- // баланс для дебетовой или сумма баллов для накопительной
        			},
        			...
        		]
        	},
        	...
        ],
        adds: [...]
    }
    
    error: {
        "code": #,
        "detail": "Сообщение об ошибке"
    }
    
    method DELETE - выход
    
    success: 'ok'
    
    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

### Регистрация/вход посредстром PIN-кода

####Генерация PIN

    URL: /v1/Pin/
    
    method POST: {
        "tel": "9128649589", // 10-значный номер телефона
        "nme": "название", // для последующей привязки к аккаунту
        "method: "sms|phone" // метод отсылки PIN кода
    }
    генерируется и отправляется PIN код на указанный номер телефона
    
    success: "ok"
    
    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Проверка PIN

    URL: /v1/Pin/:pin/
    
    method GET
    GET params:
        sendPwd: "--" // сгенерировать и выслать новый пароль
    
    success: {
		... // данные пользователя
    }

    - если пользователь новый, заводится аккаунт + привязывается указанный номер телефона, 
    - если существующий - выполняется вход,
    - если вход уже выполнен, привязывается доп. номер телефона

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

-----------------

###Справочники

####Список городов
    
    URL: /v1/Towns/
    
    method GET
    GET params:
    	nme: "--" // часть имени
    	pageSize: 999 // размер страницы
    	page: 1 // номер страницы
    
    success: [
    	{
    		id: 161,
    		nme: "Вологда"
    	},
    	{
    		id: 324,
    		nme: "Воркута"
    	},
    	...
    ]

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Список служб
    
    URL: /v1/Services/
    
    method GET
    GET params:
    	twn_id: 3, // ID города
    	nme: "--" // часть имени
    	pageSize: 999 // размер страницы
    	page: 1 // номер страницы
    
    success: [
		{
			fed_tel: "88212313131",
			trf_id: 9,
			twn_id: 3,
			twn: {
				nme: "Сыктывкар",
				id: 3
			},
			id: 250,
			rgm_id: 3,
			nme: "31-31-31"
		},    	
		...
    ]

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Список адресов
    
    URL: /v1/Adds/
    
    method GET
    GET params:
    	twn_id: 3, // ID города 
    	type: 2|9 // тип адреса
    	nme: "" // часть адреса
    	limit: ## // размер страницы MAX = 100
    
    success: [
	    {
		    nme: "Южная",
		    type: 9,
		    twn_id: 3
	    },
	    {
		    nme: "Автовокзал (Морозова 202)",
		    type: 2,
		    twn_id: 3
	    },
		...    
    ]

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Список домов 
    
    URL: /v1/Addr/
    
    method GET
    GET params:
    	twn_id: 3, // ID города 
    	type: 2|9 // тип адреса
    	nme: "" // часть адреса
    
    для улицы
    success: [
	    {
		    nme: "Южная",
		    type: 9,
		    twn_id: 3,
		    hses: [
			    {
				    id: :id, // id адреса
				    hse: :hse // номер дома
			    },
			    ...
			]
	    },
		...    
    ]
    
    для быстрого адреса
    success: [
	    {
		    nme: "Сова (Интер. 133)",
		    type: 2,
		    twn_id: 3,
		    id: :id // id адреса
	    },
		...    
    ]

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Список тарифов
    
    URL: /v1/Tariffs/
    
    GET params:
            twn_id: 3, // ID города 

    success: [
	    {
		    twn_id: :id, // id города
	        trfs: [
		        {
			        nme: :nme, // наименование
                    level: 1|2|3|4|5|6, // класс машины/обслуживания, определяет картинку на сайте
                    default: true|false, // тариф по умолчанию
			        srv_ids: [] // список id служб
		        },
		        ...
	        ]
	    },
	    ...
    ]
    или 
    {
	    twn_id: :id, // id города
        trfs: [
	        {
		        nme: :nme, // наименование
                level: 1|2|3|4|5|6, // класс машины/обслуживания, определяет картинку на сайте
                default: true|false, // тариф по умолчанию
		        srv_ids: [] // список id служб
	        },
	        ...
        ]
    }
    
    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Список текущих акций
    
    URL: /v1/Actions/
    
    GET params:
            twn_id: --, // ID города или отсутствует

    success: [
        {
            rem: "Акция 21-0000",
            trf_id: 59,
            num: "315",
            srv_id: 244,
            twn_id: 3
        },
        ...
    ]
    если twn_id передан, или:
    [
        {
            rem: "Акция 21-0000",
            trf_id: 59,
            num: "315",
            srv_id: 244,
            twn_id: 3
        },
        ...
    ]
    для всех городов, если акция глобальная, то __twn_id = null__

    __ВНИМАНИЕ!__ применение карты может влиять не только на тариф, поэтому как __srv_id__ так и __trf_id__ могут быть равными __null__
    
    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Тарификатор
    
    URL: /v1/Rater/
    
    method POST: {
        "adrs": [ // список адресов
            {
                "srt": 0,
                "twn_id": 3
                "adr": "Улица",
                "hse": "Дом", // не обязателен если "type": 2
                "ent": "Подъезд",
                "type": 9, // тип адреса 2 - быстрый, 9 - улица/дом
                "lat": 0.0, // широта 
                "lon": 0.0 // долгота 
                //при наличии координат, адрес т.е. "adr" и "hse" - не учитываются
            },
            {
                "srt": 1,
                "twn_id": 3,
                "adr": "Улица",
                "hse": "Дом", // не обязателен если "type": 2
                "ent": "Подъезд",
                "type": 2, // тип адреса 2 - быстрый, 9 - улица/дом
                "lat": 0.0, // широта 
                "lon": 0.0 // долгота 
                //при наличии координат, адрес т.е. "adr" и "hse" - не учитываются
            },
            ...
        ],
        "trf_id: --, // ID тарифа
        "twn_id": --, // ID города
        "more": 1|0, // возвращаем или нет поле adrs в ответе
        "strict": 1|0, // если точное совпадение адреса не найдено, то при strict=1 выдаст ошибку, иначе попытается найти похожий
        "srv_id": --, // <id службы>, - если не пришёл тариф и город, то определение этих параметров произойдёт по службе
        "dist_km": --, // <пробег в километрах с дробной частью> - не будет пытаться определить трек, а возьмёт данные по длине трека из запроса
        "tme": -- // <строка в формате YYYY-MM-DDTHH:MM:SS> - определит стоимость не на текущий момент, а на указанное время (временная зона по серверу)
    }
    
    success: {
	    "dist_km": 4.5,
	    "trf_id": 57,
	    "twn_id": 3,
	    "wtd_cost": 160
	}
    
    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

-------------------------

###Работа с заказами

####Заведение заказа
    
    URL: /v1/Orders/
    
    POST: {
        "tel": "9128880011",
        "adds": [
            {
                "srt": 0,
                "twn_id": 3
                "adr": "Улица",
                "hse": "Дом", // не обязателен если "type": 2
                "ent": "Подъезд",
                "type": 9, // тип адреса 2 - быстрый, 9 - улица/дом
                "lat": 0.0, // широта 
                "lon": 0.0 // долгота 
            },
            {
                "srt": 1,
                "twn_id": 3,
                "adr": "Улица",
                "hse": "Дом", // не обязателен если "type": 2
                "ent": "Подъезд",
                "type": 2, // тип адреса 2 - быстрый, 9 - улица/дом
                "lat": 0.0, // широта 
                "lon": 0.0 // долгота 
            },
            ...
        ],
        "options": [], // дополнительные опции заказа (пока не определены)
        "crd_num": "5911" // номер карты,
        "crd_type": 0, // тип карты 0 - не использовать, 3 - дебетовая, 8 - накопительная (возможны и другие)
        "srv_id": 259, // ID - службы
        "type": 0, // тип заказа 0 - срочный, 1 - предварительный
        "tme_drv": "2014-12-01T10:27:06.858Z", // время заказа ISO 8601 (для предварительного заказа)
        "dist_km": ###, // ожидаемая дистанция поездки (из тарификатора)
        "wtd_cost": ###, // ожидаемая стоимость поездки 
    }
    
    success: {
        "id": 89345 // ID заказа
    }

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

    GET - получение списка текущих заказов

    success: [
    	{
    		"id": ----, // ID заказа
    		"tel": "----------",
    		"adds": [
    			...
    		],
    		"crd_num": "---",
    		"crd_type": 0, // тип карты 0 - не использовать, 3 - дебетовая, 8 - накопительная (возможны и другие)
    		"options": [], // дополнительные опции заказа (пока не определены)
    		"srv_id": 259, // ID - службы
    		"type": 0, // тип заказа 0 - срочный, 1 - предварительный
    		"tme_reg": "2014-12-01T10:27:06.858Z", // время регистрации заказа ISO 8601
    		"tme_drv": "2014-12-01T10:27:06.858Z", // время заказа ISO 8601
    		"wtd_cost": --- // ожидаемая стоимость заказа
    	},
    	...
    ]

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####Статус заказа
    
    URL: /v1/Orders/:id/

    GET - получение статуса заказа

    success: {
		"srv_reg_id": 259, // ID - службы регистрации
		"srv_exe_id": 259, // ID - службы выполнения
		"state": - // числовой статус заказа 0-6 
		"drv_lat": ---, // широта автомобиля
		"drv_lon": ---, // долгота автомобиля
		"tme_reg": "---", // время регистрации заказа
		"tme_drv": "---", // время подачи такси (для предварительного заказа)
		"tme_wait": "---", // время прибытия такси
		"auto_rem": "н505ке11 / красный Шевроле Лачетти", // назначенный автомобиль
        "tme_drv_avg": ###, // среднее время назначения водителя на заказ
        "tme_reg_period": ### // секунд прошло со времени регистрации заказа (время сервера)
	}

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

    DELETE - удаление (отказ от заказа)

    success: "ok"

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }

####История заказов
    
    URL: /v1/ArcOrders/

    GET - получение списка заказов
    GET params:
    	tme_bgn: "2014-12-01T10:27:06.858Z", // время заказа ISO 8601 (default now - 1 week)
    	tme_end: "2014-12-02T10:27:06.858Z", // время заказа ISO 8601 (default now) !!! максимальный период 6 недель
    	pageSize: 999 // размер страницы
    	page: 1 // номер страницы

    success: {
		"id": ----, // ID заказа
		"tel": "----------",
		"adds": [
			...
		],
		"options": [], // дополнительные опции заказа (пока не определены)
		"crd_num": "---",
		"crd_type": 0, // тип карты 0 - не использовать, 3 - дебетовая, 8 - накопительная (возможны и другие)
		"srv_reg_id": 259, // ID - службы регистрации
		"srv_exe_id": 259, // ID - службы выполнения
		"type": 0, // тип заказа 0 - срочный, 1 - предварительный
		"state": - // числовой статус заказа 0-6 
		"tme_reg": "---", // время регистрации заказа
		"tme_drv": "---", // время подачи такси
		"auto_rem": "н505ке11 / красный Шевроле Лачетти", // назначенный автомобиль
		"cost": --- // стоимость заказа
	}

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }


####Трек архивного заказа
    
    URL: /v1/ArcOrders/:id/Track/

    GET - получение трека архивного заказа

    success: [
    	{
    		"lat": ---,
    		"lon": ---,
    		"tme_db": "---"
    	},
    	...
    ]

    error: {
        "code": 1,
        "detail": "Сообщение об ошибке"
    }


----------

#Сценарии

1. получение справочников

        GET: /v1/Towns/
        GET: /v1/Services/
        GET: /v1/Tariffs/
2. проверка регистрации, данных пользователя (телефоны, адреса, статус, карты и т.д.)

        GET: /v1/Users/

###Новый пользователь

1. ввод данных о заказе, обращения к справочникам адресов

        GET: /v1/Adds/
        GET: /v1/Addr/
2. расчёт стоимости поездки

        POST: /Rater/

3. проверка номера телефона (неявная регистрация)

        POST: /v1/Pin/
        GET: /v1/Pin/:pin/
4. заведение заказа

        POST: /v1/Orders/
5. периодическая проверка статуса заказа

        GET: /v1/Orders/:id/

###Существующий пользователь

1. получение списка архивных заказов для извлечения адресов предыдущих поездок

        GET: /v1/ArcOrders/
2. ввод данных о заказе, обращения к справочникам адресов

        GET: /v1/Adds/
        GET: /v1/Addr/
3. расчёт стоимости поездки

        POST: /Rater/
4. заведение заказа

        POST: /v1/Orders/
5. периодическая проверка статуса заказа

        GET: /v1/Orders/:id/
