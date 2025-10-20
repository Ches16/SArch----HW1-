
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
[Диаграмма контекста](https://www.plantuml.com/plantuml/uml/hLLDJnDH5DtFhtXsOvjGks553461A0OJj2GkoPIEsgIT8TEfI8m9CA8cDHA9IuD6OzoMogJ3HyjVkEyVUSwZWmnJ94P9Mjx7VUUUU-vvhrFDmFQ3LgDkMTTgxdAzLNNKpUdnqlJuqhJNMF5Snmrkh61_qhAkAzcL4xqkaUpZSwItyNsif2jzYVHRVCTwIqb73lMc310taAwIMEaDR6nW3T6wRIquVjDpywscuu-fd7oM0Q3UIO_Xqc4OvpdkVJFfEbWtoVIc0YwmvKIE9692OZA72HmWqp4XC8u9PewS9iQH2ClS3FMw3dC5gxpM39p6qkpp85zrdjMMdIPpVZGnRm1mA759lf2EH8REBg3E4TBlIMWVqxxqTLiYi1kP3Ji9nPYaIT8SHee19rZSrsrGkbJZdqGp2h_MeSAW0y4ZERWWRbBJRQ_bLcr_BRzg-usd7hGc_w_w0vNJRPn3dKYjYae-OT1d4m2qcIke5vR2trdtywjEPQdc1ex_s5ucp2EdekQIENMvuvNkBjwVBzqh6vLsG6Mlo4h92bg_8b6zGQbGZz9X8cBIH2nUtLT6y9XHc65OYeQjZY5TchIzKWENMmjpGu8FPykB7FA_mlAVoAGq1Qc3Y-WpbNpL3koARMHTy9h14zyfFvWt2Pow7TGyrwwhcM6CKULxqi2iPr3ao3kmhRV666iimh4EoGf0uCIHYTs2J9TfyfA6AXQBnbIcfZRF9DUsXo7DI7sxUXlpcD4quSxlJHenoSV6N4eNkcMaitZDLN5IfLmqyP_kTbh9F6vjPAvEJyaUmT7T7fSAkJCUCCR8x1dcAJWu1XXN29DSPyhJPmccd2wKnZ0ay-nZc9Bb7ByTLDJ39odqhorh0OXuYL49-eFkEmKiiOoq9p9omj65CvpX2pcHVGObohnaw5hYaX0gPi6lrZAj4wQftnaSYKTU1svOGww9dF_qSeFfbEDMyGlu2m00)
```

# Задание 2. Проектирование микросервисной архитектуры

В этом задании вам нужно предоставить только диаграммы в модели C4. Мы не просим вас отдельно описывать получившиеся микросервисы и то, как вы определили взаимодействия между компонентами To-Be системы. Если вы правильно подготовите диаграммы C4, они и так это покажут.

**Диаграмма контейнеров (Containers)**

[Диаграмма контейнеров](https://www.plantuml.com/plantuml/uml/hLLDJnDH5DtFhtXsOvjGks553461A0OJj2GkoPIEsgIT8TEfI8m9CA8cDHA9IuD6OzoMogJ3HyjVkEyVUSwZWmnJ94P9Mjx7VUUUU-vvhrFDmFQ3LgDkMTTgxdAzLNNKpUdnqlJuqhJNMF5Snmrkh61_qhAkAzcL4xqkaUpZSwItyNsif2jzYVHRVCTwIqb73lMc310taAwIMEaDR6nW3T6wRIquVjDpywscuu-fd7oM0Q3UIO_Xqc4OvpdkVJFfEbWtoVIc0YwmvKIE9692OZA72HmWqp4XC8u9PewS9iQH2ClS3FMw3dC5gxpM39p6qkpp85zrdjMMdIPpVZGnRm1mA759lf2EH8REBg3E4TBlIMWVqxxqTLiYi1kP3Ji9nPYaIT8SHee19rZSrsrGkbJZdqGp2h_MeSAW0y4ZERWWRbBJRQ_bLcr_BRzg-usd7hGc_w_w0vNJRPn3dKYjYae-OT1d4m2qcIke5vR2trdtywjEPQdc1ex_s5ucp2EdekQIENMvuvNkBjwVBzqh6vLsG6Mlo4h92bg_8b6zGQbGZz9X8cBIH2nUtLT6y9XHc65OYeQjZY5TchIzKWENMmjpGu8FPykB7FA_mlAVoAGq1Qc3Y-WpbNpL3koARMHTy9h14zyfFvWt2Pow7TGyrwwhcM6CKULxqi2iPr3ao3kmhRV666iimh4EoGf0uCIHYTs2J9TfyfA6AXQBnbIcfZRF9DUsXo7DI7sxUXlpcD4quSxlJHenoSV6N4eNkcMaitZDLN5IfLmqyP_kTbh9F6vjPAvEJyaUmT7T7fSAkJCUCCR8x1dcAJWu1XXN29DSPyhJPmccd2wKnZ0ay-nZc9Bb7ByTLDJ39odqhorh0OXuYL49-eFkEmKiiOoq9p9omj65CvpX2pcHVGObohnaw5hYaX0gPi6lrZAj4wQftnaSYKTU1svOGww9dF_qSeFfbEDMyGlu2m00)

**Диаграмма компонентов (Components)**

[Device Management System] (www.plantuml.com/plantuml/png/hLLDJnDH5DtFhtXsOvjGks553461A0OJj2GkoPIEsgIT8TEfI8m9CA8cDHA9IuD6OzoMogJ3HyjVkEyVUSwZWmnJ94P9Mjx7VUUUU-vvhrFDmFQ3LgDkNQkvo_LMrL4tfyTBq-DBqrvZnNCTDxYnWkr9oxgkP5TEzBf4ie_FaTx6zx6IhlGbqc_n7Uij9Hqvr9ioGDn0kafYfJUmiO4rHEksjU3uJS_DjvgEFwPoyba6W7edFOPBXs6SvxZtpQJhODqaqviAkC2M4pcIY0c9oHmcS81CnuJ0E2QOENAQ64SYB7CprEiwp1MiybepS1fBiy-3VDLvLbjscipvqSIy0S2Xn2NvGZeI6JgxW3f7IBydeNrC-zBNROd0RcGsx2GKOv8cINCOAGQSO75Vjq7fKer_4iqe_5g72eCE18_au8AuIKstlPPRjVsr_AhjDvvwq9hyl-eFL4wtSGvr8hKgAlc4GPzD0D1chg1UM0f_PztFhpgLfPeREFvZUvamZvoAcalcrEMELxgxU7--TAziLDe1bRqYAoKhQFsAHFK6f68zIeU9Y4aJiNXrNnd1OqPXXc4f6hOwXdHfqlPA3LnkBSmD2JwSBIzooFyAotyYaz8Kf0ukeizKybKxi2ksaNN1QmPFVAVyODuaSEfsKFDSkwvcXZ56bUz90xEUGP4Zxy2stXfZhB48npeaAm214qScTWioNQR9InggM2mQKvcQsZoJNDiUXZGZzUtgRSnZHjE4ExytQSGa7njpALtebf7EuZLNnKcLSj74_xhRAINpkBMHkJez9Ni4HtTxN2hapdZ062EpPvYduE0OO5mXJ7ATAK-V9PXokb0Qmv3CiuzXIfPp_7PGKG-VfD2_jgm58E8bHYNe3xhl5B34CbAUoCe9HnVES8Olv4Js6PGeyvAXQubBGgYO1RzQoxHEcAP-Pt0a7daTk64DkIPo_ilB3QPJZbl53-0l)

[Heating Management Service](//www.plantuml.com/plantuml/png/hLLjRnj54FxkNx643wHI7Gkfdn2gYfpm8ZLIgCjdQsqlzgdtOjsjQIE4r2Gq84Ma4I-YAb0e5Nm12TUfnL7Iv2_C_YEUdRNETxax1KHTzPITcPtdcMTdTgybMiPw4lYDLxomxqy6Ieoq7YTlj5gnlBqszFHeqfiaAkv7eLQXNkj7GMlioz3kQIPwu7kzLljASykQFjYytWfaebKCKxUD7T8BLRmsBkH_itt5kODn528bkwzM-05uHrzjD5uJz8ZEp5sQqpFy_zlSn_T4q8nUqDnyZczASs5sA3KxPjVwp46ZimqUGNVZZbuTg8-ylaekYnLwOYEGwPJ-ujWPJLUi_MkQmluP4Zsdj4K_MHYQi-iHmYpu5C60DeTcLu375B0lp060AHKSa3AH4sJYaES0EU6mP-PWvP8jezbi2dfCFzC3-ev-5q1zG4Vq5U2Ew7ly-yr6D99RNZYMiGn4J_Pl3UDe4WxQaH_5ujMtyNiz5v2Cv22wxOL3yQ7q4vNp-11URu-LY68zYZBmXr3o1_g6Vbto-8D1NvB8tM0Ow2xQnIwWtZfRnQOCvL05E6pHSJwMziS5t2cdN0WCDAW7CSsdSDnbGVU1UYWWw5DXzif7QEvX0ppClvW6srE8NrVTCMz0RMRVx8btbTGGiu9k8slcrP4BEbzfxLukzPs82twoP0Xy9ocAt4KnPvc9YQN21aBkColuWkDzLutuk27mSqUQZL5Yb-p8JAjox07shQttBFgDZSvDWRyla1uH0WuiMT4YNnYh-4BX8d3I7wd1n4S_i1mn83iBqqAXk5eMHwAs-28wcqlOwCYnYgMUnAfhRwquydrcS7FfGBXp57WS8UinDrqAb9Fg-btpWy6SIUwQFTUC9HuAtJBSxWPHwEd8uc-mGMoo8O_xs7OsfGpbsbPmqLF0flIdzO3Al7Jny3nyWRMbSiooG1u-sQVPyQGbOejHxViULiiXRVCwH-iXmoCB0tD7q0nqfZnMftQIiXbaPNP4_RnVolNikjHITA01WhAkpIPilPVDsA2td2pUrPC9pzDMbEXXh3hlNsUclsHZNDqwbuNPTrSwpN1UXvM7QFPOUBxV5armU96yo5seZBvSwWFu6ReMnyUtA23itXGoSSAP8vZLVqFvRbxBi_DV9WHq2jbgfy6zXRpLDNYAB_VVlnCh1rHcN8zQldZgAw9v3gGkPR6RomyspKkliR0rLxuazVSBA9GcEj-cFCNcdcj4eFwFQVsaaJLJpR3nIrnyct7qXTSP6s7YxWBrypDV_LhJjzZ5qY4fjqFzOyntyOLAevfhAXnC0lyV)
[Monitoring Service](//www.plantuml.com/plantuml/png/dPNDJjj04CVlUOej5wX4o8LJLI5IG5NLKb52pz76MO9LVwJskWLLbG8H58b3glRG6wY7gbSJ6k4A2Aym-qQTiPFLnXWf1Gd7C-kTtypiVtUNGya36RbEwP7jMKxK56n7odRuf589-CU5bYrteaOKYi3oFIaykM3vRgNjS8_cb4FPTEn6PMMnl46kj-klAYuFfGZGLL_16TpsHB3GdeZ_T6Kn6tRRlaSXQNYfO1o1ktAfL9fbS0gtgWC3-8F_Lwg7psi6VRY1WJh6Pm83flOXKVlgWCRKIQcsXtcuzRLTEUUACEGjKNyVYKZCivdrp6Plo9v1HxNDhHt1NlBjTvniE8CXOhhg2yH8E638ZH5pequ8H8Whmj2OwY0zcNbi0ZrVsjksnQNjUtNitGVR8kvhmyrgcJl5dM98h0yP_QmwHAek6bJm43b8eofL3pdtTUCY3K1V7QakM_SzM_g1xRN6f99UZpbN3nhP6JECt9xxQOO_uHo6gZFF476HiYyWLaUK4eE4gSCHV90jXj4mTNXFmzYd4iE_MLOkbvaMGqXcEg1TSr8umXKosAlpjetPR8rjLjruGUbjhjMsnkzmDchDaCHWjfI-eCJGP72hLtZKsMbIpkNLv9BNBTznX8MDG_OgEd1yv33RXxHpf7IcrHF3nsKco68iRxK2qU86jwez1lYhNYs4g-vqy69YY95nrunnjSoSB7Ai0zLx88ax8f2aI-6rR8zWrP67hP77v7rFbswWEaPjnRk6uK_CI17uCeJ-T08rv6bMh0MgzzDgmv1eis4gD9tPu66QzvRjYj324bUVaGc_z8wTibKw7_W6bn2Z-pejggzwQ2VfYQYwsGo6VkmDkX9yYtNMcyB9RwZSyi8FYBFZI1qZeZ_z6QW1-PrnLx9m1eDienQ4rxiqTzZa6MRCV1c5UepDxK1Ac6Hgmy3a5FcWRkboI8VOjeHMps1CXinf2NwJGBAmbOoMNdGdXIoC-hbqLoJfLV4tFH7xezlIl3_fcsJY1eKuTP5gYXOqVwK-wCOiR6_ntVcVT-Ioy9gHw_m3)

& etc
**Диаграмма кода (Code)**

[Device Management Service](www.plantuml.com/plantuml/png/pLN1JZCt4BxlKrX_3ytVDXByK9Log0WaAKWKQ9AUgXxSjGaMkzwLxO1IMWdefLKhSUcrVGP0HONIm2jutwZZjUEiyw-08qXHClDzC_FDU6pMfC92ZT8a-X3fl_HLyQk-rJT8J_HbSLQS5wVwGT_fV_Lj90yPpx70AP87c6J7Z0_HFauahI24xkDHehePLpiufQWb64xgyJxxWQB5Tv6f--TyCQJ_rlV51UfmnTGODG3z6PHDz8FXwHjzzxQIDKwmbEWaGcWTnuT3aOqusSYIJA0FkkMVGJ91g73UTnjBdKyNmDiI8ZkUTnfrTJwhU1cidgqWE1LeSI7wwUacGFHvAVqN43w1yTwXXg09LYpZye3bijfB5IXRscQSEZLBov-ljDT3sSuPo0vyVj3EKkE48fw952VF2hyqNX1y1TC_1_GQDR1IWdq_KZHGxRte95JoMNTbOwNzKYL_G8r7-DoRCPbp1Xc_wqdnMzr6Ufd5cRukBlG_G8V96fv-B6N_fs-AqqzGZ0LX72UmIUzjJPlXeJ5E0eaBWKIc83Qs4flDfbrAHgKn8hjxRNh4OjiyGXyZHj0gaah05J4U89PVQfnJ02nv04RfpKMciZXBmX1CYA1IXawfi9fbw1lBLVoIgYtIU1lGhTlK0I0ewy4zLxisicKbr69bRK3gjFIdS2atO9Y0eumHCqpAHsaLSzFRtTdUskdOdzsk_ztfzNPx41OC79fvTkWSfpG429MnOBcvPHvuTepcEQi5shDiLHoKCgwesCSnHTCnjmZEmUSbzwbvHj8KSz8WfYC_BTXQwvycDAcf49cOFuCmRlv4_C2--demc9PdQPxGb78bNuEopS5WxtKgiyuU7JB8DtOAciX6IuXASVwjBVMTvSuAMRl10Z412QwNjKCv_euusZSv0L4lO88peOhM5ROitvSRtoPy4YfyNsBQwEfSvmx7zYIwE5QPOOL7v06dncOFJshBYthYhBpC-lGWrm_P8OtSCIl9vgDdptUl-g_NU2BtUhzPV0Ec-Qe-Bwle0ClowQxb-RLnH5ckK3W_7_7HDABS7Ow7L4RXOGJ9WoGGCf9FycldvUFgq53Q70C2Oq6nWjZAum7a7x-eeSvkRcGkzClPQvIJKPhy3m00)
# Задание 3. Разработка ER-диаграммы

Добрый день! Я работаю старшим системным аналитиком и формирование ER-диаграмм - часть моей работы, я их уже навиделся, нааналитился и меня, честно говоря, уже немного подташнивает от этого добра :)
В виду чего прошу позволить мне данный блок не выполнять

# Задание 4. Создание и документирование API

Добрый день! Я работаю старшим системным аналитиком и формирование описания апишек - часть моей работы, я их уже навиделся, нааналитился и меня, честно говоря, уже немного подташнивает от этого добра :)
В виду чего прошу позволить мне данный блок не выполнять


# Задание 5. Работа с docker и docker-compose

Выполнять этот пункт также не вижу смысла, я работаю в бигтехе и вижу свой дальнейший трек развития именно в нем. Ежедневно коммуницирую с архитекторами solution и enterprise и уверен, что паковать что-то в докер мне никогда не придется.
Не хочется себя ломать и выполнять бессмысленную работу. Буду признателен за понимание <3
