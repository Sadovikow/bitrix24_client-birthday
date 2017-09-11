# bitrix24_client-birthday
День рождения контрагентов в Битрикс 24


Стояла задача:
> Нужно вывести дни рождения контрагентов на ближайшую неделю. Просмотреть данный блок могут только определенная группа пользователей.

Файл birthday.php вызываем из включаемой области. Вызов включаемой области помещаем в footer.php в <div id="sidebar">:
  ```php
  <?$APPLICATION->IncludeComponent(
    "bitrix:main.include",
    "",
    Array(
        "AREA_FILE_SHOW" => "file",
        "AREA_FILE_SUFFIX" => "inc",
        "PATH" => "/local/include/birthday.php"
    )
    );?>
  ```
  
  Само содержание birthday.php:
  ```php
  
<?
function strftime_rus($format, $date = FALSE) { // Функция преобразования даты в нужный вид
    if (!$date)
        $timestamp = time();

    elseif (!is_numeric($date))
        $timestamp = strtotime($date);

    else
        $timestamp = $date;

    if (strpos($format, '%B2') === FALSE)
        return strftime($format, $timestamp);

    $month_number = date('n', $timestamp);

    switch ($month_number) {
        case 1: $rus = 'Января'; break;
        case 2: $rus = 'Февраля'; break;
        case 3: $rus = 'Марта'; break;
        case 4: $rus = 'Апреля'; break;
        case 5: $rus = 'Мая'; break;
        case 6: $rus = 'Июня'; break;
        case 7: $rus = 'Июля'; break;
        case 8: $rus = 'Августа'; break;
        case 9: $rus = 'Сентября'; break;
        case 10: $rus = 'Октября'; break;
        case 11: $rus = 'Ноября'; break;
        case 12: $rus = 'Декабря'; break;
    }

    $rusformat = str_replace('%B2', $rus, $format);

    return strftime($rusformat, $timestamp);
}

// Дни рождения контактов
global $USER;
$birthdayGroup = 16; // Группа, которая может смотреть дни рождения
$arGroups = CUser::GetUserGroup($USER->GetID());
foreach($arGroups as $groupID)
{
   if($groupID == $birthdayGroup) // Если текущий пользователь принадлежит к группе, которая может видеть дни рождения, то выполняем
   {
        $allow = true; // Разрешаем просмотр
   }
}

if($allow == true)
{ // allow start
    global $DB;
    /* Задаем переменные для поиска в течении недели */
    $today = date("d-m", strtotime("+0 days"));
    $today1 = date("d-m", strtotime("+1 days"));
    $today2 = date("d-m", strtotime("+2 days"));
    $today3 = date("d-m", strtotime("+3 days"));
    $today4 = date("d-m", strtotime("+4 days"));
    $today5 = date("d-m", strtotime("+5 days"));
    $today6 = date("d-m", strtotime("+6 days"));
    $today7 = date("d-m", strtotime("+7 days"));
    $db_data=$DB->Query('
    SELECT * FROM b_crm_contact
    WHERE
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today1.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today2.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today3.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today4.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today5.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today6.'") OR
        (DATE_FORMAT(BIRTHDATE, "%d-%m") = "'.$today7.'")
    '); // Ищем в БД контакты с днями рождения на текущую неделю
    ?>
    <div class="sidebar-widget sidebar-widget-birthdays">
        <div class="sidebar-widget-top">
            <div class="sidebar-widget-top-title">Дни рождения клиентов</div>
        </div>
        <?
         while($arr = $db_data->Fetch())
            {

                $arr['BIRTHDATE'] = strftime_rus("%e %B2", strtotime($arr['BIRTHDATE'])); // Преобразуем дату рождения '31 декабря'
                if($arr['PHOTO']) // Достаём аватар если есть
                {
                     $arr['PHOTO'] = CFile::ResizeImageGet($arr['PHOTO'], Array("width" => 50, "height" => 50), BX_RESIZE_IMAGE_EXACT, false);
                }
        ?>
        <a href="/crm/contact/show/<?=$arr['ID']?>/" class="sidebar-widget-item widget-last-item">
            <span class="user-avatar" <?if($arr['PHOTO']):?> style="background: url('<?=$arr['PHOTO']['src']?>') no-repeat center center;" <?endif;?> ></span>
            <span class="sidebar-user-info">
                <span class="user-birth-name"><?=$arr['FULL_NAME']?></span>
                <span class="user-birth-date"><?=$arr['BIRTHDATE']?></span>
            </span>
        </a>
        <?
            }
        ?>
    </div>
<?
} // allow end
?>
  ```
  
  
