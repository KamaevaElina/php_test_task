# sql_test_task

## Задача №1

Имеется база со следующими таблицами:
```sql
CREATE TABLE `users` (

`id` INT(11) NOT NULL AUTO_INCREMENT,

`name` VARCHAR(255) DEFAULT NULL,

`gender` INT(11) NOT NULL COMMENT '0 - не указан, 1 - мужчина, 2 - женщина.',

`birth_date` INT(11) NOT NULL COMMENT 'Дата в unixtime.',

PRIMARY KEY (`id`)

);

CREATE TABLE `phone_numbers` (

`id` INT(11) NOT NULL AUTO_INCREMENT,

`user_id` INT(11) NOT NULL,

`phone` VARCHAR(255) DEFAULT NULL,

PRIMARY KEY (`id`)

);
```
Напишите запрос, возвращающий имя и число указанных телефонных номеров девушек в возрасте от 18 до 22 лет. Оптимизируйте таблицы и запрос при необходимости.

## Решение
Таблица:
```sql
CREATE TABLE `users` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(255) DEFAULT NULL,
	`gender` INT(11) NOT NULL COMMENT '0 - не указан, 1 - мужчина, 2 - женщина.',
	`birth_date` DATE NOT NULL,
	PRIMARY KEY (`id`)
);

CREATE TABLE `phone_numbers` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`user_id` INT(11) NOT NULL,
	`phone` VARCHAR(255) DEFAULT NULL,
	PRIMARY KEY (`id`),
	FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
);
```
Запрос:
```sql
SELECT 	u.id, u.name, COUNT(pn.phone)
FROM	 	users u 
LEFT JOIN	phone_numbers pn 
ON		u.id = pn.user_id
WHERE	u.gender = 2 AND 
		(TIMESTAMPDIFF(YEAR, u.birth_date, CURDATE())) BETWEEN 18 AND 22
GROUP BY u.id;
```

## Задача №2 

Имеется строка: https://www.somehost.com/test/index.html?param1=4&param2=3&param3=2&param4=1&param5=3

Напишите функцию, которая:

1. удалит параметры со значением “3”;

2. отсортирует параметры по значению;

3. добавит параметр url со значением из переданной ссылки без параметров (в примере: /test/index.html);

4. сформирует и вернёт валидный URL на корень указанного в ссылке хоста.

В указанном примере функцией должно быть возвращено: https://www.somehost.com/?param4=1&param3=2&param1=4&url=%2Ftest%2Findex.html

## Решение
```php
function convert_url($url)
{
    $parsed = parse_url($url);
    $params = explode('&', $parsed["query"]);
    foreach ($params as $key => $value) {
        $params[$key] = explode('=', $value);
        if ($params[$key][1] == '3')
            unset($params[$key]);
    }
    function cmp($arr1, $arr2)
    {
        return (strnatcmp($arr1[1], $arr2[1]));
    }
    usort($params, "cmp");
    $new_url = $parsed["scheme"] . "://" . $parsed["host"] . "/" . "?";
    foreach ($params as $key => $value) {
        $new_url .= $value[0] . "=" . $value[1] . "&";
    }
    if ($parsed["path"]) {
        $new_url .= "url=" . urlencode($parsed["path"]);
    } else {
        $new_url = substr($new_url, 0, -1);
    }
    return ($new_url);
}
```
## Задача №3

Проведите рефакторинг, исправьте баги и продокументируйте в стиле PHPDoc код, приведённый ниже (таблица users здесь аналогична таблице users из задачи №1). Примечание: код написан исключительно в тестовых целях, это не "жизненный пример" :)

```php
function load_users_data($user_ids) {

$user_ids = explode(',', $user_ids);

foreach ($user_ids as $user_id) {

$db = mysqli_connect("localhost", "root", "123123", "database");

$sql = mysqli_query($db, "SELECT * FROM users WHERE id=$user_id");

while($obj = $sql->fetch_object()){

$data[$user_id] = $obj->name;

}

mysqli_close($db);

}

return $data;

}

// Как правило, в $_GET['user_ids'] должна приходить строка

// с номерами пользователей через запятую, например: 1,2,17,48

$data = load_users_data($_GET['user_ids']);

foreach ($data as $user_id=>$name) {

echo "<a href=\"/show_user.php?id=$user_id\">$name</a>";

}
```
Плюсом будет, если укажете, какие именно уязвимости присутствуют в исходном варианте (если таковые, на ваш взгляд, имеются), и приведёте примеры их проявления.

## Решение
```php
/**
 * @param string $user_ids separated by ,
 * @return array id => name
 */
 
function load_users_names($user_ids) {
    $db = mysqli_connect("localhost", "root", "123123", "database");
    $user_ids = explode(',', $user_ids);
    $sanitized_ids = implode(',', array_map('intval', $user_ids));
    $sql_result = $db->query("SELECT id, name FROM users WHERE id IN '(" . $sanitized_ids . "')");
    $data = array();
    while ($obj = $sql_result->fetch_object()){
        $data[$obj->id] = $obj->name;
    }
    $sql_result->close();
    $db->close();
    return $data;
}
```
Не соблюдается принцип минимизации привелегий (работа под пользователем root). Пароль для доступа к базе слишком простой и распространённый. Если в исходный скрипт подставляются данные, полученные от пользователя, то он уязвим для sql инъекций.
