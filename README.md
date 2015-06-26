# itcod-user
SOA ITCOD  

Service "USER"  Management registration USER/USERBOX for ITCOD-DISK 

Назначение: Обеспечить сервис регистрации новых пользователей для WEBDAV 
сервиса itcod-disk. Включает функции проверки и контроля корректности процесса.


-- Copyright (c) 2015 by Yura Vdovytchenko (max@itcod.com)
-- Copyright (c)itcod 2010-2015

local description = {
  _COPYRIGHT   = "Yura Vdovytchenko",
  _VERSION     = "itcod-user.lua 15.06.26",
  _URL         = "https://ihome.itcod.com/max/projects/itcod-user/",
  _DESCRIPTION = [[
   Registration userbox for WEBDAV ITCOD structure directory
   Test computation in Lua (5.1)
   Author by Yura Vdovytchenko (max@itcod.com)
  ]],
  _LICENSE = [[
    MIT LICENSE

    Permission is hereby granted, free of charge, to any person obtaining a
    copy of this software and associated documentation files (the
    "Software"), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:

    The above copyright notice and this permission notice shall be included
    in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
    OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
    IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
    CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
    TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
    SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
  ]]
}
-----------------------------------------------------------------------------------
--path lua file: /etc/nginx/lua/itcod-user.lua
--Example Nginx virtual example.conf
local example = {
  _NGINX = [[
server {
    #...

    location / {
	#...
    }

    location /user/ {
	content_by_lua_file /etc/nginx/lua/itcod-user.lua;
    }
}
  ]]
}

ВАЖНОЕ ПОЯСНЕНИЕ ЛОГИКИ РАБОТЫ ПРОГРАММЫ

1.
Пользователь и Юзербокс itcod-диска являются синонимами при регистрации 
но отличаются применением. Поэтому в документе эти термины используются в зависимости от 
логики применения.

Юзербокс это одноименная имени пользователя папка на диске, владельцем (администратором) 
которой изначально назначается регистрируемый пользователь. То есть при регистрации пользователя
USER в корне диска будет создана папка USER и в ней помещён файл .htpasswd в 
котором первой строкой будет зарегистрирован пользователь USER (администратор юзербокса
всегда на первой строке. поставите другого на первую строку и он станет администратором 
и имя юзербокса будет отличаться от имени администратора(на данный момент не рекомендуется так как
может заблокировать различные функции этой программы и иных)). 

2.
Юзербокс может распологаться не только в корне диска itcod, но и в любом другом юзербоксе
или вложенной папке любого юзербокса где установлены разрешения регистрации 
пользователей. При этом параметр userbox в запросе примет значение пути к папке регистрации.
userbox=SUPERUSER/USER или например userbox=SUPER/buper/USER

3.
Рекомендуется регистрироваться англицскими символами и цифрами. Не рекомендуется 
применение спецсимволов и национальных кодировок.


ОБЯЗАТЕЛЬНЫЕ УСЛОВИЯ УСПЕШНОГО ВЫПОЛНЕНИЯ СЕРВИСОМ РЕГИСТРАЦИИ ЮЗЕРБОКСА

1. Наличие в папке где будет создан юзербокс файла разрешения регистрации .htreg
2. Отсутствие файла блокировки имени юзербокса (userbox.lock)
3. Корректное формирование запроса

КЛЮЧИ ЗАПРОСОВ

Обязательные для любого запроса
userbox - логин регистрируемого пользователя (юзербокса)
cmd - комманда для сервиса (test/add, в развитии del/lock/unlock/newpw)

Необходимые для определённых типов запросов
npw1 - новый пароль
npw2 - новый пароль (повтор)
opw - старый пароль

ТИПЫ ЗАПРОСОВ (cmd)

test - проверка наличия пользователя (юзербокса)
add - регистрация пользователя (юзербокса)
del - удаление пользователя [в разработке]
lock - блокировка доступа к юзербоксу [в разработке]
unlock - разблокировка доступа к юзербоксу [в разработке]
newpw - смена пароля [в разработке]

ОБЯЗАТЕЛЬНЫЕ ПАРАМЕТРЫ В ЗАПРОСАХ
test - userbox cmd
add - userbox cmd npw1 npw2 (npw12 - пароли админа)
del - userbox cmd npw1 npw2
lock - userbox cmd npw1 npw2
unlock - userbox cmd npw1 npw2
newpw - userbox cmd npw1 npw2 opw

ОТВЕТЫ СЕРВИСА

Передаются в теле ответа (body) на отдельной строке между тегами
<!-- START REPLY --> 
ответ статуса
<!-- STOP REPLY -->

status=ok - регистрация выполнена полностью
status=usr.none - пользователь не найден (можно регистрировать)
status=usr.found - пользователь существует
status=usr.block - пользователь заблокирован (наличие одноименного пользователя/сервиса или запрет)
status=err.sys - ошибка в процессе обработки запроса
status=err.userbox - ошибка в имени пользователя(юзербокса)
status=err.param - ошибка в параметрах запроса (например отсутствие обязательных)
status=err.cmd - ошибка команды (отсутствие или неизвестна)
status=err.passwd - ошибка пароля (например отсутствие или несовпадение пары)


ПРИМЕРЫ ЗАПРОСОВ
https://ihome.itcod.com/user/?userbox=USER&cmd=test
https://ihome.itcod.com/user/?userbox=USER&cmd=add&npw1=PASSW&npw2=PASSW

ПРИМЕР ОТВЕТА

<!-- START REPLY --> 
status=usr.none
<!-- STOP REPLY -->


HISTORY

15.06.26 StartUp версия


РАЗВИТИЕ

1. Добавить комманды del/lock/unlock/newpw
2. Добавить системы потверждения регистрации (защита от авторегистраций)
3. Добавить блокировку регистрации по локальным словарям и внешним сервисам
