# Знакомство с gossip

## Push

В этой реализации мы только рассылаем другим узлам имеющуюся у нас информацию. Проблема - как остановить рассылку, не обладая глобальной информацией о том, получили ли все узлы информацию. Мы её пока игнорируем используя флаг `-q`. 

```
$ cargo run -- -i ..\push.py -n 1000 -f 2 -q

Nodes: 1000
Fanout: 2
Network drop rate: 0
Implementation: ..\push.py

time       delivered    stopped      messages
1          1            0            2
2          3            0            8
3          9            0            26
4          26           0            78          
5          75           0            228
6          201          0            630
7          466          0            1562        
8          795          0            3152        
9          963          0            5078        
10         991          0            7060        
11         999          0            9058        
12         1000         0            11058 

Messages sent by each node: max=24, min=2, mean=11.06
```

## Pull

В этой реализации мы только запрашиваем у других узлов имеющуюся у них информацию. Узел прекращает запрашивать информацию (становится `stopped`) после её получения. Здесь проблемы остановки нет.

```
$ cargo run -- -i ..\pull.py -n 1000 -f 2

Nodes: 1000
Fanout: 2
Network drop rate: 0
Implementation: ..\pull.py

time       delivered    stopped      messages
1          1            1            1998
2          4            4            3993        
3          12           12           5977        
4          32           32           7933        
5          95           95           9806        
6          253          253          11470       
7          594          594          12686       
8          925          925          13316       
9          999          999          13452
10         1000         1000         13453

Messages sent by each node: max=20, min=6, mean=13.45
```

## Push + Pull

В этой реализации мы комбинируем предыдущие подходы, обмениваясь информацией в обе стороны. Это опять привносит проблему остановки, мы её решим в следующей реализации.

```
$ cargo run -- -i ..\push_pull.py -n 1000 -f 2 -q

Nodes: 1000
Fanout: 2
Network drop rate: 0
Implementation: ..\push_pull.py

time       delivered    stopped      messages
1          1            0            2000        
2          11           0            4008        
3          91           0            6070        
4          463          0            8405        
5          966          0            11650       
6          1000         0            15618

Messages sent by each node: max=27, min=12, mean=15.62
```

## Push + Pull + Stop Condition

Ранее для реализаций с push мы останавливали симуляцию в тот момент, когда информация достигла всех узлов. Но поскольку сами узлы про это ничего не знают, то наши реализации продолжат сплетничать бесконечно. Один из способов остановить push-based gossip без глобальной синхронизации между узлами -- отказаться с некоторой вероятностью от дальнейшей отправки сообщений узлом, если он получил уже имеющуюся у него информацию. Теперь мы можем честно выполнить симуляцию до конца, когда все сообщения в системе закончатся.

```
$ cargo run -- -i ..\push_pull_stop.py -n 1000 -f 2

Nodes: 1000
Fanout: 2
Network drop rate: 0
Implementation: ..\push_pull_stop.py

time       delivered    stopped      messages
1          1            0            2000
2          11           0            4008        
3          91           3            6064        
4          471          79           8230        
5          958          636          10074       
6          1000         972          10838
7          1000         999          10896       
8          1000         1000         10897

Messages sent by each node: max=19, min=6, mean=10.90
```

## Оптимизации

Какие оптимизации можно реализовать, чтобы уменьшить число рассылаемых сообщений?

## Влияние потерь сообщений

Ранее мы запускали симулятор с надежной сетью, где все сообщения доставлялись. Запустите теперь симулятор с опцией `-d 0.2`, когда 20% сообщений теряются. Как изменились результаты работы разных реализаций?