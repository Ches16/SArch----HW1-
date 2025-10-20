
# Задание 1. Анализ и планирование


## 1. Описание функциональности монолитного приложения

**Управление отоплением:**

*Действие:* Пользователь может отправлять команды на включение или выключение системы отопления.
*Механизм:* Синхронный запрос от сервера к устройству. Сервер выступает инициатором.

**Мониторинг температуры:**

*Действие:* Пользователь может просматривать актуальную температуру в доме через веб-интерфейс.
*Механизм:* Синхронный запрос от сервера к датчику для получения данных (pull-модель). Сервер периодически опрашивает датчик.

## 2. Анализ архитектуры монолитного приложения

Текущая архитектура является классическим монолитом, что было оправданным решением на старте, но стало главным препятствием для масштабирования и реализации новых бизнес-целей.

- **Стек**: Go, PostgreSQL.
- **Архитектурный стиль**: Монолит.
- **Взаимодействие**: Синхронное (request-response).

### Сильные стороны (на начальном этапе):
- **Простота разработки**: Единая кодовая база и процесс сборки упрощают начальную разработку.
- **Простота развертывания**: Один артефакт для развертывания.
- **Отсутствие сетевых задержек**: Все вызовы происходят внутри одного процесса.

### Слабые стороны (текущие проблемы и риски):
- **Низкая масштабируемость**: Невозможно масштабировать отдельные компоненты. При росте нагрузки на получение телеметрии придется масштабировать все приложение, включая редко используемые функции.
- **Технологическая связанность**: Вся система привязана к одному стеку (Go, PostgreSQL). Внедрение новых технологий для специфических задач (например, NoSQL для временных рядов телеметрии) затруднено.
- **Высокая стоимость внесения изменений**: Любое изменение, даже незначительное, требует полной пересборки и развертывания всего приложения, что повышает риск регрессионных ошибок.
- **Низкая отказоустойчивость**: Сбой в одном компоненте (например, ошибка при обработке данных с датчика) может привести к отказу всей системы.
- **Неэффективная модель взаимодействия с устройствами**: Постоянный опрос (pull-модель) сотен, а в будущем тысяч устройств, создаст колоссальную нагрузку на сервер и сеть. IoT-системы требуют push-модели, где устройства сами отправляют данные при их изменении.

## 3. Определение доменов и границы контекстов

Применительно к дорабатываемой системе, можно выделить следующие доменты и границы их контекстов:

### Управление Устройствами:
- **Ответственность**: Отвечает за управление устройствами в системе, включая добавление, удаление, настройку и мониторинг активности. Этот домен будет отвечать за взаимодействие с датчиками и реле.
- **Граница контекста**: Включает в себя все компоненты, связанные с управлением устройствами, такие как API для управления устройствами, логику для отправки команд на устройства, хранилище данных об устройствах и их состоянии.

### Управление Отоплением:
- **Ответственность**: Отвечает за управление системой отопления, включая установку целевой температуры, создание расписаний и мониторинг энергопотребления.
- **Граница контекста**: Включает в себя компоненты, специфичные для управления отоплением, такие как алгоритмы управления отоплением, API для управления отоплением, хранилище данных о настройках отопления.

### Мониторинг:
- **Ответственность**: Отвечает за сбор, обработку и отображение данных телеметрии с устройств.
- **Граница контекста**: Включает в себя компоненты, связанные со сбором данных, такие как API для приема данных телеметрии, логику для обработки и агрегации данных, хранилище данных телеметрии и интерфейсы для отображения данных.

### Пользователи и Дома:
- **Ответственность**: Отвечает за управление пользователями, их домами и правами доступа.
- **Граница контекста**: Включает в себя компоненты, связанные с аутентификацией, авторизацией, управлением профилями пользователей, информацией о домах и связями между пользователями и домами.
- 
## **4. Проблемы монолитного решения**

## В текущей реализации ПО свойственны все недостатки монолитного решения:

1. **Ограниченная масштабируемость и устойчивость**  
   Если количество пользователей, активно использующих функцию мониторинга температуры, резко возрастет, это может перегрузить весь монолит, даже если другие функции (например, управление отоплением) не испытывают повышенной нагрузки. Более того, если в модуле управления отоплением возникнет ошибка, это может привести к недоступности и сервиса мониторинга.

2. **Сложность развертывания и обновления**  
   Если команда разработчиков исправит небольшую ошибку в веб-интерфейсе, потребуется пересобрать и переразвернуть весь монолит, включая backend-логику и базу данных. Это может занять несколько часов и потребовать остановки сервиса, что вызовет недовольство пользователей. Кроме того, это требует координации всех пяти разработчиков и трех тестировщиков.

3. **Технологический застой и сложность внедрения новых технологий**  
   Если компания захочет использовать новую базу данных, более подходящую для хранения данных телеметрии, потребуется переписать значительную часть кода, связанного с доступом к данным, во всем монолите. Это потребует больших усилий и времени, а также может привести к нестабильности системы. Кроме того, разработчики, работающие с Go и PostgreSQL, могут не захотеть изучать новую базу данных.

4. **Проблемы организационного характера и проектного управления.**

## 5. Визуализация контекста системы — диаграмма С4

```markdown
[Диаграмма контекста]([https://www.plantuml.com/plantuml/uml/hLLDJnDH5DtFhtXsOvjGks553461A0OJj2GkoPIEsgIT8TEfI8m9CA8cDHA9IuD6OzoMogJ3HyjVkEyVUSwZWmnJ94P9Mjx7VUUUU-vvhrFDmFQ3LgDkMTTgxdAzLNNKpUdnqlJuqhJNMF5Snmrkh61_qhAkAzcL4xqkaUpZSwItyNsif2jzYVHRVCTwIqb73lMc310taAwIMEaDR6nW3T6wRIquVjDpywscuu-fd7oM0Q3UIO_Xqc4OvpdkVJFfEbWtoVIc0YwmvKIE9692OZA72HmWqp4XC8u9PewS9iQH2ClS3FMw3dC5gxpM39p6qkpp85zrdjMMdIPpVZGnRm1mA759lf2EH8REBg3E4TBlIMWVqxxqTLiYi1kP3Ji9nPYaIT8SHee19rZSrsrGkbJZdqGp2h_MeSAW0y4ZERWWRbBJRQ_bLcr_BRzg-usd7hGc_w_w0vNJRPn3dKYjYae-OT1d4m2qcIke5vR2trdtywjEPQdc1ex_s5ucp2EdekQIENMvuvNkBjwVBzqh6vLsG6Mlo4h92bg_8b6zGQbGZz9X8cBIH2nUtLT6y9XHc65OYeQjZY5TchIzKWENMmjpGu8FPykB7FA_mlAVoAGq1Qc3Y-WpbNpL3koARMHTy9h14zyfFvWt2Pow7TGyrwwhcM6CKULxqi2iPr3ao3kmhRV666iimh4EoGf0uCIHYTs2J9TfyfA6AXQBnbIcfZRF9DUsXo7DI7sxUXlpcD4quSxlJHenoSV6N4eNkcMaitZDLN5IfLmqyP_kTbh9F6vjPAvEJyaUmT7T7fSAkJCUCCR8x1dcAJWu1XXN29DSPyhJPmccd2wKnZ0ay-nZc9Bb7ByTLDJ39odqhorh0OXuYL49-eFkEmKiiOoq9p9omj65CvpX2pcHVGObohnaw5hYaX0gPi6lrZAj4wQftnaSYKTU1svOGww9dF_qSeFfbEDMyGlu2m00])
```

# Задание 2. Проектирование микросервисной архитектуры

В этом задании вам нужно предоставить только диаграммы в модели C4. Мы не просим вас отдельно описывать получившиеся микросервисы и то, как вы определили взаимодействия между компонентами To-Be системы. Если вы правильно подготовите диаграммы C4, они и так это покажут.

**Диаграмма контейнеров (Containers)**

[Диаграмма контейнеров](https://www.plantuml.com/plantuml/uml/hLLDJnDH5DtFhtXsOvjGks553461A0OJj2GkoPIEsgIT8TEfI8m9CA8cDHA9IuD6OzoMogJ3HyjVkEyVUSwZWmnJ94P9Mjx7VUUUU-vvhrFDmFQ3LgDkMTTgxdAzLNNKpUdnqlJuqhJNMF5Snmrkh61_qhAkAzcL4xqkaUpZSwItyNsif2jzYVHRVCTwIqb73lMc310taAwIMEaDR6nW3T6wRIquVjDpywscuu-fd7oM0Q3UIO_Xqc4OvpdkVJFfEbWtoVIc0YwmvKIE9692OZA72HmWqp4XC8u9PewS9iQH2ClS3FMw3dC5gxpM39p6qkpp85zrdjMMdIPpVZGnRm1mA759lf2EH8REBg3E4TBlIMWVqxxqTLiYi1kP3Ji9nPYaIT8SHee19rZSrsrGkbJZdqGp2h_MeSAW0y4ZERWWRbBJRQ_bLcr_BRzg-usd7hGc_w_w0vNJRPn3dKYjYae-OT1d4m2qcIke5vR2trdtywjEPQdc1ex_s5ucp2EdekQIENMvuvNkBjwVBzqh6vLsG6Mlo4h92bg_8b6zGQbGZz9X8cBIH2nUtLT6y9XHc65OYeQjZY5TchIzKWENMmjpGu8FPykB7FA_mlAVoAGq1Qc3Y-WpbNpL3koARMHTy9h14zyfFvWt2Pow7TGyrwwhcM6CKULxqi2iPr3ao3kmhRV666iimh4EoGf0uCIHYTs2J9TfyfA6AXQBnbIcfZRF9DUsXo7DI7sxUXlpcD4quSxlJHenoSV6N4eNkcMaitZDLN5IfLmqyP_kTbh9F6vjPAvEJyaUmT7T7fSAkJCUCCR8x1dcAJWu1XXN29DSPyhJPmccd2wKnZ0ay-nZc9Bb7ByTLDJ39odqhorh0OXuYL49-eFkEmKiiOoq9p9omj65CvpX2pcHVGObohnaw5hYaX0gPi6lrZAj4wQftnaSYKTU1svOGww9dF_qSeFfbEDMyGlu2m00)

**Диаграмма компонентов (Components)**

[Device Management System] (//www.plantuml.com/plantuml/png/hLRTRXj55BxtKmpXWaQAiqBg5K9LIr8e4f524eJK6sjiBywgtbrhTv8sGaX9IfiYIsbKG5Jyf1G4N8349etRRNushp3pHdnpPcjFCcie55jUUSyvCzyvt_cPkPmAdeYLi5jxBOXQtPMspvQ5wALlULx2Rqvt0h6yqbn9_QGLHyAFn7Gh3hrUbqTwJJqLxMxGz6OkrkUrwAEv3xoGfy9F86hCO0KF8Z-PxftP_-sPYqOTzk88MvBwKeKUWBV4fLhj3IPtvQbQbmDvWD-Hke_dCPFxyaGEr3qySpbWQaFcQaDjQftgrnRNu4VOk7fBN6Zxgq7BJwVOXFnPMs2dGZudstsPJMZvLpA3V0iR7ShSapzg63aWrIxCD7W6Oq2hRRN9u4S6s1FL1x3C6HdavCYnp8p98K2EoUn0zIScTHZrUft9n_8d-L1-9_zWG7qeT-INWDkHt-FxkxQefJU2gCSJ7h8cRzte9F5Ar9w9kt72NdyNdpSjWtIPj-ERGTHXd_1kwbkQBeWNQpsVnObOZeVWtuB97-JNyjUH7tzgpNYIdr3eY4pTLRVPB579vdZ4Etw8vB55Fz6YsZ3R9TsDi1EAHcgqu77iIatoU_44PMT4DC3BjDvsKmPUyocG3jc1zXRQf-0xCtaeO5GG_KVuxKzCiazhDOQFuM1OTwb-9SLGT87t0ZXoPVvzxS_2rSKbLlwNZqfg2dAYB0P0RseB1WLUuUpv24XO7G5guYmuMYlaKRknZ4VNJu0wOoJicf5OLEtAFGBQeyBjqvu3f-nUqI7C1e54KigqEulwlKnXukJDvaHjb4GL402LQid5TH3BsadyJf2AP6rKDmkbn4BzPLYP21lDfpu7sZeru5q8jnbmzYg3LNSSu5uIYxWLTnkypNk2efqlHUoA4LdGEpG0Jh5RhjR1COMilNacZo3QCfF1AKVLht27NPZxS6bfYbrRMfgVOjU3JjFtzKnHczFoO7fo74Nj86t5gxxDqUmBqTaw0KKPUVbKviCwWH2zGLvMCOJXgdCuq16Eym2KhGPf44UOFoCdvYtfEJy6sF4KpotR3uZ1MW551hNsQl2Yim_1s3Qinuy2k6PrzYmNd2sAE45vMhrTpgNPvf2wfiKPrZHvIbDeBeuwyUmx_qDnsLx91uWlevC1BG7Ex-CzfzCBo_NYt52z3b6EVzdGzmM_QoQIEnda0-JksQZnpIb6GuZIUahZxsJOy_PSe8IQtPsDpxTyMK35t_Ps9GyMohbDtIQNZx7P7KEvGV7a8nq1DjwY0Fzb5WP0jnYinFo6MXYOwXuLvAl1k5Gw8yMzJ5IDRDskLLW5fVWtAXIq3AL15gJ2BiO8oxRL5mZXkJDxB4_T2UAoGrUW9p8tRbXZhYo6eZhhXGjZZucNoRrplIgORY0Od7E8_20xpTy_fSS1jeS1Q4MZcmhmJ64GKl-_byLdjPhR_tIrAgtAIwjrZIMR_Cn5rXpCD11q46v9bNVRbwmjkggE-ieXcwOWJYDF7qQUEOkyw_MF-QgFa2xxKHjNzRy0)

[22](//www.plantuml.com/plantuml/png/hLLjRnj54FxkNx643wHI7Gkfdn2gYfpm8ZLIgCjdQsqlzgdtOjsjQIE4r2Gq84Ma4I-YAb0e5Nm12TUfnL7Iv2_C_YEUdRNETxax1KHTzPITcPtdcMTdTgybMiPw4lYDLxomxqy6Ieoq7YTlj5gnlBqszFHeqfiaAkv7eLQXNkj7GMlioz3kQIPwu7kzLljASykQFjYytWfaebKCKxUD7T8BLRmsBkH_itt5kODn528bkwzM-05uHrzjD5uJz8ZEp5sQqpFy_zlSn_T4q8nUqDnyZczASs5sA3KxPjVwp46ZimqUGNVZZbuTg8-ylaekYnLwOYEGwPJ-ujWPJLUi_MkQmluP4Zsdj4K_MHYQi-iHmYpu5C60DeTcLu375B0lp060AHKSa3AH4sJYaES0EU6mP-PWvP8jezbi2dfCFzC3-ev-5q1zG4Vq5U2Ew7ly-yr6D99RNZYMiGn4J_Pl3UDe4WxQaH_5ujMtyNiz5v2Cv22wxOL3yQ7q4vNp-11URu-LY68zYZBmXr3o1_g6Vbto-8D1NvB8tM0Ow2xQnIwWtZfRnQOCvL05E6pHSJwMziS5t2cdN0WCDAW7CSsdSDnbGVU1UYWWw5DXzif7QEvX0ppClvW6srE8NrVTCMz0RMRVx8btbTGGiu9k8slcrP4BEbzfxLukzPs82twoP0Xy9ocAt4KnPvc9YQN21aBkColuWkDzLutuk27mSqUQZL5Yb-p8JAjox07shQttBFgDZSvDWRyla1uH0WuiMT4YNnYh-4BX8d3I7wd1n4S_i1mn83iBqqAXk5eMHwAs-28wcqlOwCYnYgMUnAfhRwquydrcS7FfGBXp57WS8UinDrqAb9Fg-btpWy6SIUwQFTUC9HuAtJBSxWPHwEd8uc-mGMoo8O_xs7OsfGpbsbPmqLF0flIdzO3Al7Jny3nyWRMbSiooG1u-sQVPyQGbOejHxViULiiXRVCwH-iXmoCB0tD7q0nqfZnMftQIiXbaPNP4_RnVolNikjHITA01WhAkpIPilPVDsA2td2pUrPC9pzDMbEXXh3hlNsUclsHZNDqwbuNPTrSwpN1UXvM7QFPOUBxV5armU96yo5seZBvSwWFu6ReMnyUtA23itXGoSSAP8vZLVqFvRbxBi_DV9WHq2jbgfy6zXRpLDNYAB_VVlnCh1rHcN8zQldZgAw9v3gGkPR6RomyspKkliR0rLxuazVSBA9GcEj-cFCNcdcj4eFwFQVsaaJLJpR3nIrnyct7qXTSP6s7YxWBrypDV_LhJjzZ5qY4fjqFzOyntyOLAevfhAXnC0lyV)
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()


**Диаграмма кода (Code)**

Добавьте одну диаграмму или несколько.

# Задание 3. Разработка ER-диаграммы

Добавьте сюда ER-диаграмму. Она должна отражать ключевые сущности системы, их атрибуты и тип связей между ними.

# Задание 4. Создание и документирование API

### 1. Тип API

Укажите, какой тип API вы будете использовать для взаимодействия микросервисов. Объясните своё решение.

### 2. Документация API

Здесь приложите ссылки на документацию API для микросервисов, которые вы спроектировали в первой части проектной работы. Для документирования используйте Swagger/OpenAPI или AsyncAPI.

# Задание 5. Работа с docker и docker-compose

Перейдите в apps.

Там находится приложение-монолит для работы с датчиками температуры. В README.md описано как запустить решение.

Вам нужно:

1) сделать простое приложение temperature-api на любом удобном для вас языке программирования, которое при запросе /temperature?location= будет отдавать рандомное значение температуры.

Locations - название комнаты, sensorId - идентификатор названия комнаты

```
	// If no location is provided, use a default based on sensor ID
	if location == "" {
		switch sensorID {
		case "1":
			location = "Living Room"
		case "2":
			location = "Bedroom"
		case "3":
			location = "Kitchen"
		default:
			location = "Unknown"
		}
	}

	// If no sensor ID is provided, generate one based on location
	if sensorID == "" {
		switch location {
		case "Living Room":
			sensorID = "1"
		case "Bedroom":
			sensorID = "2"
		case "Kitchen":
			sensorID = "3"
		default:
			sensorID = "0"
		}
	}
```

2) Приложение следует упаковать в Docker и добавить в docker-compose. Порт по умолчанию должен быть 8081

3) Кроме того для smart_home приложения требуется база данных - добавьте в docker-compose файл настройки для запуска postgres с указанием скрипта инициализации ./smart_home/init.sql

Для проверки можно использовать Postman коллекцию smarthome-api.postman_collection.json и вызвать:

- Create Sensor
- Get All Sensors

Должно при каждом вызове отображаться разное значение температуры

Ревьюер будет проверять точно так же.


