# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе # выполнил(а):
- Барчанинов Иван Николаевич
- РИ-220947

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

## Цель работы
научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выберите одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры. Выберите одну из игровых переменных в игре (ресурсы, внутри игровая валюта, здоровье персонажей и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.
**Assassin’s Creed: Brotherhood** — компьютерная игра в жанре action-adventure и стелс-экшена с видом от третьего лица, разработанная компанией Ubisoft Montreal.
![image](https://github.com/D9eka/DA-in-GameDev-2/assets/120364200/82a82998-f4f0-4162-b3c0-6723ff129a43)

Игровой процесс стандартен для серии Assassin’s Creed и является развитием игрового процесса Assassin's Creed II. Как и в предыдущих частях, основой механики игры является паркур, использование толпы и различных укрытий для незаметных убийств и развитой системы ближнего боя.

Одним из главный аспектов экономики в Assassin’s Creed: Brotherhood служит восстановление города: 
> **ЗАВОЮЙТЕ СЕРДЦЕ ГОРОДА:** используйте ваши с трудом заработанные деньги, чтобы восстановить полуразрушенный город. Сплотите граждан к вашей натуре и разблокируйте новые фракции и миссии.
![1_0CTkdscMnQ7-5ejOM-kXiw](https://github.com/D9eka/DA-in-GameDev-2/assets/120364200/41d39a90-aca7-4a55-8917-6330a8cf095c)

В качестве игровой переменной я выбрал **процент восстановления города**. Переменная изменяется, когда игрок восстанавливает одно из зданий в городе. Диапазон значений: от 0 до 100 процентов.

Схема экономической модели:

![Group 6](https://github.com/D9eka/DA-in-GameDev-2/assets/120364200/c2816e09-71ea-456e-bc6c-c6b2e454f9c5)


## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в выбранной игре (в качестве таких переменных может выступать игровая валюта, ресурсы, здоровье и т.д.). Средствами google-sheets визуализируйте данные в google-таблице (постройте график, диаграмму и пр.) для наглядного представления выбранной игровой величины.

Скрипт для _Python_ будет генерировать количество денег, а после высчитывать, сколько процентов города можно восстановить, если потратить на это все деньги. На восстановление города нужно потратить примерно _200 000_, от этой суммы и будет высчитываться процент.

```py

import gspread
import numpy as np
gc = gspread.service_account(filename='da-in-gamedev-2-a92933725ef8.json')
sh = gc.open("DA-in-GameDev-2")
money_for_restoration = 200000
price = np.random.randint(10000, 200000, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon) - 1:
    i += 1
    if i == 0:
        continue
    else:
        restoration_percent = round((price[i] / money_for_restoration) * 100, 2)
        restoration_percent = str(restoration_percent)
        restoration_percent = restoration_percent.replace('.',',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(restoration_percent))
        print(restoration_percent)

```

## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

Скрипт для Unity будет запускать звуковые сигналы, в зависимости от процента восстановление города:
1. Меньше 25% - bad speak;
2. В промежутке от 25% до 75% - normal speak;
3. Больше 75% - good speak.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class GoogleSheetsController : MonoBehaviour
{
    [SerializeField] private AudioClip badSpeak;
    [SerializeField] private AudioClip normalSpeak;
    [SerializeField] private AudioClip goodSpeak;

    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    private void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    private void Update()
    {
        if(dataSet.Count == 0)
            return;

        if (dataSet["Mon_" + i.ToString()] < 25 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 25 & dataSet["Mon_" + i.ToString()] <= 75 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 75 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1DXr_F4tzujM2XSaE6TjYJh7Hh8Kw_Lio4XVDpMHuMD0/values/Лист1?key=AIzaSyC5XCKRGFbxalcIt38IN5kBlIeb0ITZwmg");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[1]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }

    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}
```

## Выводы

В процессе выполнения лабораторной работы я научился передавать в Unity данные из Google Sheets с помощью Python, а также познакомился с https://console.cloud.google.com .

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
