# Atomic Swap between Omni Layer's currencies and other cryptocurrencies
# Atomic Swap на Omni Layer
 Предположим, что Алиса и Боб желают совершить межблокчейновый обмен криптовалют - Алиса хочет обменять ***a*** единиц какой-либо Omni
 валюты, например TetherUS (валюта имеет идентификатор валюты #31 в Mainnet, далее в тексте будем говорить только об этой валюте протокола
 Omni, так как она на данный момент она является самой популярной, но приведенный ниже алгоритм будет работать также для любой валюты
 протокола Omni) на ***b*** единиц криптовалюты работающей на другом блокчейне (Напомним, что Omni работает поверх блокчейна биткоина,
 конечно, по приведенному ниже алгоритму можно произвести обмен TetherUS на биткоины, но в силу их работы на одном блокчейне такой обмен
 может быть совершен другим, более эффективным способом).
## Обозначения
  ***A*** - блокчейн биткоина. <br />
  ***B*** - блокчейн той криптовалюты на которую производится обмен TetherUS. <br />
  ***a*** - сумма TetherUS, которую Алиса хочет обменять. <br />
  ***b*** - сумма криптовалюты исполюзующей блокчейн ***B***, на которую Алиса хочет обменять свои ***a*** TetherUS. <br />
##  Создание транзакции

   1)Боб генерируетслучайное значение `secret`.<br />
   
   2)Боб высчитывает `secretHash` проведя следующую операцию: `secretHash = RIPEMD160( secret )` <br />
   
   3)Боб создает и передает в блокчейн ***B*** htlc-транзакцию, закрытую значением `secretHash` <br />
   
   4)Боб передает Алисе значение `secretHash`, и хэш созданной им в предыдущем пункте htlc-транзакции, для того чтобы Алиса могла убедиться
   в том, что в блокчейне ***B*** действительно присуствует нужная htlc-транзакция. <br />
   
   5)Алиса получив от Боба `secretHash` и хэш созданной Бобом htlc-транзакции, убеждается в том, что такая транзакция действительно
   находится в блокчейне ***B***, и в том, что это действительно htlc-транзакция закрытая значением `secretHash`.
   
   6)Алиса используя полученный `secretHash` создает следующую транзакцию и транслирует её в блокчейн биткоина:
   
   ![Funding_tx](https://github.com/swaponline/tether.research/blob/master/images/1.png)
   
   назовем такую транзакцию `funding_tx`, по факту это почти обычная биткоиновая htlc-транзакция которая используется в atomic swap с той лишь разницей, что в поле amount 546 сатошей - это минимальное количество биткоинов которое может быть на выходе транзакции,
   ниже этого значение протокол биткоина считает транзакцию пылью(dust) и не проводит её.
   
   7)Алиса создает транзакцию по следующей схеме:
   
   ![Redeem_tx](https://github.com/swaponline/tether.research/blob/master/images/2.png)
   
   назовем такую транзакцию `redeem_tx`. Алиса создает такую транзакцию с двумя входами: первый - это вход ссылающиеся на выход `funding_tx`
   который содержит htlc скрипт этот вход Алиса не подписывает, то есть поле SigScript остается вообще пустым. второй вход - это вход 
   ссылающиеся на любой непотраченый выход Алисы, главное условия чтобы на этом выходе было достаточное количество биткоинов для оплаты
   transaction fee, этот вход Алиса подписывает своим приватным ключом обязательно с типом подписи [SIGHASH_ALL](https://bitcoin.org/en/glossary/sighash-all)(то есть подписывает всю 
   транзакцию кроме полей SigScript на входах транзакции, что делает эту транзакцию неизменяемой. выходы же транзакции представляют
   собой обыкновенный Simple Send ***a*** TetherUS от Алисы Бобу(подробнее о том, что такое Simple Send, payload и как это
   работает в другом разделе). 
   
   8)Алиса отправляет Бобу созданную в предыдущем пункте и подписанную собой `redeem_tx`.
   
   9)Боб получив отправленную Алисой `redeem_tx`, проверяет её - просто просматривает входы, и выходы, убеждается в том, что это
   действительно транзакция которую должна была создать Алиса используя настоящий алгоритм. После чего Боб подписывает транзакцию своим
   приватным ключом и предоставляет значение `secret` в SigScript соответствующего входа `redeem_tx`.
   
   10)Боб транслирует подписанную собоё транзакцию `redeem_tx` в блокчейн, тем самым производя перевод валюты TetherUS от Алисы себе.
   **Примечание** - перед проведением этого шага, еще нужно проверить, что на адресе Алисы действительно есть необходимая сумма TetherUS.
   
   11)Алиса просматривая блокчейн ***A*** получает значение `secret` и используя его в блокчейне ***B*** переводит себе средства с
   созданной Бобом в пункте 3 htlc-транзакции. Обмен на этом завершается.
   
   **Очевидное примечание: естественно значение timelock используемое Бобом при создание htlc-транзакции должно быть значительно больше
    timelock который использует Алиса, так как её htlc-транзакция должна тратиться первее чем htlc созданный Бобом.
    Это необходимо для того, чтобы Боб не смог успеть потратить оба htlc**
# Что такое payload, Simple Send и как с ними работать?
 ## Что такое Omni Layer?
  Omni - это протокол, работающий над блокчейном биткоина,  позволяющий всем желающим сконструировать свою собственную валюту(чем-то
  похоже на токены эфира, только со в значительной степени меньшими возможностями).
 ## Как Omni проводит транзакции
  Для проведения транзакции Omni нужно создать обычную биткоин транзакцию-перевод 546 сатошей(минимум) с дополнительным выходом - хранящим payload посредством оп-кода [OP_RETURN](https://en.bitcoin.it/wiki/OP_RETURN), [Пример такой транзакции](https://www.blockchain.com/ru/btc/tx/f5cf2bcfc3fa46d10140cc1a4940ae9eb00803d6f79fb4c7a159e8befaf1cf71).  Payload - это обязательная часть любой Omni транзакции, является последовательностью байтов, содержащей всю информацию об транзакции.
  
  Рассмотрим, какую информацию хранит в себе payload: <br />
  1. **transaction marker** - 4 байта, обязательная часть любого Omni payload, всегда равна `0x6f6d6e69` - ASCII код `omni`. Если первые 4 байта последовательности не равны `0x6f6d6e69`, то эта последовательность не является payload Omni. В [коде](https://github.com/swaponline/tether.research/blob/master/function/createRedeemTransaction.cpp#L57) смотреть строки 62-65. <br />
  2. **version** - 2 байта, аналог версии транзакции в биткоине. Для работы описанного алгоритма, используется версия `0`, или что то же самое `0x0000`. Omni. В [коде](https://github.com/swaponline/tether.research/blob/master/function/createRedeemTransaction.cpp#L57) смотреть строки 68-69. <br />
  3.**transaction type** - 2 байта, тип транзакции, для проведения atomic swap достатояно использовать только "Simple send" транзакции, simple send - это обычное отправление omni валюты со своего адреса на адрес получателя. simple send соответствует коду типа транзакции `0` то есть следующие 2 байта `0x0000`. [другие возможные типы транзакции существующие в омни](https://github.com/OmniLayer/spec#field-transaction-type). Omni. В [коде](https://github.com/swaponline/tether.research/blob/master/function/createRedeemTransaction.cpp#L57) смотреть строки 72-73.<br />
  4.**token identifier** - 4 байта, идентификатор используемой валюты, Например TetherUS имеет идентификатор `31` или `0x0000001f`. Все созданные протоколом Omni на данный момент времени токены можно увидеть по следующей ссыки - https://www.omniexplorer.info/properties/production . Omni. В [коде](https://github.com/swaponline/tether.research/blob/master/function/createRedeemTransaction.cpp#L57) смотреть строки 76-79.<br />
  5. **amount** - 8 байт, для транзакции типа Simple send, это количество отправляемой валюты. Omni. В [коде](https://github.com/swaponline/tether.research/blob/master/function/createRedeemTransaction.cpp#L57) смотреть строки 82-86.
  
  Как можно заметить, payload не хранит в себе адреса отправителей и получателей транзакции, эти адреса протокол определяет по биткоиновой транзакции в которой был обнаружен выход с payload'ом. Omni протокол просматривая входы определяет кто производит перевод, найдя среди входов транзакции p2pkh выход соответствующего адресу отправите. Если у транзакции несколько p2pkh входов, то оправителем считается адрес чей индекс входа минимальный. По аналогии определяется получая, просматривая уже все выходы транзакции, протокол находит выход p2pkh на адрес, этот адрес и считается адресом получателем omni-транзакции.
  
  Таким образом для передачи от Алисы к Бобу, например 50,000,000 TetherUS, нужно создать биткоин транзакцию один из входов которой будет ссылаться на p2pkh выход соответствующий адресу Алисы, так же важно чтобы этот вход был первым в этой транзакции (индекс этого входа в полученной транзакции был бы минимален или вообще равен нулю). Один из выходов этой транзакции должен быть выход p2pkh на адрес Боба, и еще один из выходов был быть выход со следующей payload:
  
   ![Payload](https://github.com/swaponline/tether.research/blob/master/images/payload.png)

Пример такой транзакции:https://www.blockchain.com/ru/btc/tx/1f359902f666249b73f7bf8a1ea778259fa4b4eb904538e9713c487e091d4756,  https://www.omniexplorer.info/search/1f359902f666249b73f7bf8a1ea778259fa4b4eb904538e9713c487e091d4756 .

## Примеры
### testnet
 В следующих примерах: <br />
  Адрес Алисы - `mgco3HEFvZ1iTaovhCEz1ZjF2wNR6Xu6oz`. <br />
  Адрес Боба - `mk3dDgfZPTn3aiNUFZAd3wWcDgV6S7gjAA` <br />
  
  Баланс OMNI токенов Алисы: <br />
  ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/images/balanceA1.png)
    
  Баланс OMNI токенов Бобаи: <br />
  ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/images/balanceB1.png)
  
  Алиса меняет 1 OMNI на какое-либо число какой-либо криптовалюты Боба. Для проведение обмена необходимо сделать следующие шаги: <br />
  1. Боб сгенерировал секретное значение -`0xa8de8a6fc5592d13752c3623754bb9fa1124392ffdb090e409ce56726cda42ea` <br />
  
  2. Хэш секретного значения `secretHash`=`0xf1883766dca084c36332d536e88e3966c2660251`, который Боб отправил Алисе. <br />
  
  3. Алиса создала следующую транзакию и отправила ее в блокчейн (funding_tx): <br />
  ![funding_tx_Example](https://github.com/swaponline/tether.research/blob/master/images/funding_tx.png) <br />
  Для наглядности выделем значение `secretHash` в explorer'е: <br />
  ![funding_tx_Example](https://github.com/swaponline/tether.research/blob/master/images/fundung_txSECRET.png)
  (Боб делает в блокчейне ***B*** то же самое, но нас эти транзакции сейчас не особо интересует, да и для разных блокчейнов они могут различаться) <br />
  
  4. Алиса создает следующую транзакцию(redeem_tx), подписывая свой вход SIGHASH_ALL алгоритмом:
  ![redeem_tx_without_Bob's_sign](https://github.com/swaponline/tether.research/blob/master/images/Redeem_tx1.png)<br />
  отправляет её Бобу на подпись(по канал связи между Алисой и Бобом), заметим, что на одном из входов отсуствует SigScript. <br />
  
  5.Боб подписывает предыдущую транзакцию и предоставляет `secret`, отправляет в блокчейн биткоина уже следующую транзакцию: <br />
  ![redeem_tx](https://github.com/swaponline/tether.research/blob/master/images/Redeem_tx2.png) <br />
  Для наглядности выделем значение `secret` в explorer'е: <br />
  ![redeem_tx_Example](https://github.com/swaponline/tether.research/blob/master/images/Redeem2SECRET.png)
  
  Таким образом, ввиду присуствия в транзакции, которую Боб последней транслировал в сеть выхода с payload'ом - это транзакция является  следующей Omni транзакцией: <br />
    ![omni_tx_Example](https://github.com/swaponline/tether.research/blob/master/images/OMNI_TX.png)
    
   Баланс OMNI токенов Алисы после проведения этих транзакции: <br />
   ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/images/BalanceA2.png)
    
   Баланс OMNI токенов Боба после проведения этих транзакции: <br />
   ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/images/BalanceB2.png)
   
   ### mainnet
 В следующих примерах: <br />
  Адрес Алисы - `1QCEnxXU9QAsnqfufArkiuangGNGqdE4in`. <br />
  Адрес Боба - `1341545XP8GdCiL96osVr3NefXpHbjzoCs` <br />
  
  Баланс TetherUS токенов Алисы: <br />
  
  ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/balanceA1.png)
    
  Баланс TetherUS токенов Боба: <br />
  
  ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/balanceB1.png)
  
  Алиса меняет 1 TetherUS на 0.001 биткоинов Боба. Для проведение обмена необходимо сделать следующие шаги: <br />
  1. Боб сгенерировал секретное значение -`0x832f296f8f2cff3fe553dee413d8fc84def9102be547bd147c02c36b704ebafb` <br />
  
  2. Хэш секретного значения `secretHash`=`0xbd52a5a7d6d5b8367ddfa8417c271c5639f85322`.<br />
  
  3. Боб создал следующую транзакию и отправила ее в блокчейн (funding_tx): <br />
  
  ![funding_tx_Example](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/funding_txB.png) <br />
  
  4. Боб передает `secretHash` значение Алисе.
  
  5. Алиса создала следующую транзакию и отправила ее в блокчейн (funding_tx): <br />
  
  ![funding_tx_Example](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/funding_txA.png) <br />
  
  6. Алиса создает следующую транзакцию(redeem_tx), подписывая свой вход SIGHASH_ALL алгоритмом:
  
  ![redeem_tx_without_Bob's_sign](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/redeem_txA.png)<br />
  отправляет её Бобу на подпись(по канал связи между Алисой и Бобом), заметим, что на одном из входов отсуствует SigScript. <br />
  
  7.Боб подписывает предыдущую транзакцию и предоставляет `secret`, отправляет в блокчейн биткоина уже следующую транзакцию: <br />
  
  ![redeem_tx](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/redeem_txAB.png) <br />
  
  Таким образом, ввиду присуствия в транзакции, которую Боб последней транслировал в сеть выхода с payload'ом - это транзакция является  следующей Omni транзакцией: <br />
  
  ![omni_tx_Example](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/omni_tx.png)
   
  8. Используя `secret`, который Алиса получила из redeem транзакции Боба, она создает и отправляет в сеть следующую транзакцию:
  
  ![redeem_txB](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/redeem_txB.png)
  
  Тем самым собирая себе биткоины.

   Баланс TetherUS токенов Алисы после проведения этих транзакции: <br />
  
  ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/BalanceA2.png)
    
   Баланс TetherUS токенов Боба после проведения этих транзакции: <br />
  
  ![Alice's_Balance](https://github.com/swaponline/tether.research/blob/master/exampleMainnet/BalanceB2.png)   
  
   Как мы видим Боб получил свой TetherUS, а Алиса свои биткоины!
  
