# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Гуревнин Дмитрий Романович
- 2012676
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.





## Цель работы
познакомиться с программными средствами для организции
передачи данных между инструментами google, Python и Unity

## Задание 1
- Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity. 
- В облачном сервисе google console подключить API для работы с google
sheets и google drive.
- ![image](https://user-images.githubusercontent.com/80514942/193807021-1c6a5c78-64c6-4336-a623-55dc0496e728.png)
- Реализовать запись данных из скрипта на python в google-таблицу. Данные
описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с
учётом стоимости игрового объекта в каждый период.
```py

import gspread
import numpy as np
gc = gspread.service_account(filename= 'our-brand-364417-9a30a1f46b8d.json')
sh = gc.open("UnitySheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1, 11))
i = 0
while i <= len(mon):
    i += 1
    if i ==  0:
        continue
    else:
        tempInf = ((price[i-1]-price[i-2])/price[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.',',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i-1]))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(tempInf)

```
- ![image](https://user-images.githubusercontent.com/80514942/193807880-cee88180-adf0-48f2-9e53-b12944e48af2.png)
- Создать новый проект на Unity, который будет получать данные из google-
таблицы, в которую были записаны данные в предыдущем пункте.
-![image](https://user-images.githubusercontent.com/80514942/193808016-0913c5bf-dbaa-4c42-a873-27c69e865474.png)
-Написать функционал на Unity, в котором будет воспризводиться аудио-
файл в зависимости от значения данных из таблицы.
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1AsH1Z6uEysjAwXACRI2P4fhmjFNiuIq83tPdBO8McUA/values/Лист1?key=AIzaSyDnnwIH-R5dmOoDzYUD3EDGlTq7shDdKUU");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
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


## Задание 2
Реализовать запись в Google-таблицу набора данных, полученных
с помощью линейной регрессии из лабораторной работы № 1
-В таблицу заносим значение ошибки итераций из первой лабароторной работы
```py
import gspread
import numpy as np
gc = gspread.service_account(filename= 'our-brand-364417-9a30a1f46b8d.json')
sh = gc.open("UnitySheets")
a = np.random.rand(1)
b = np.random.rand(1)

Lr = 0.000001

for i in range(10):
    a, b = iterate(a, b, x, y, 100 * (i + 1))

    prediction = model(a, b, x)
    loss = loss_function(a, b, x, y)

    sh.sheet1.update(('A' + str(i + 1)), str(100 * (i + 1)))
    sh.sheet1.update(('B' + str(i + 1)), str(a[0]))
    sh.sheet1.update(('C' + str(i + 1)), str(b[0]))
    sh.sheet1.update(('D' + str(i + 1)), str(loss))
```


## Задание 3
### Самостоятельно разработать сценарий воспроизведения звукового
сопровождения в Unity в зависимости от изменения считанных данных в задании 2
-Будем выводить разницу между ошибками итераций
```py
a = np.random.rand(1)
    b = np.random.rand(1)

    Lr = 0.000001

    old_loss = 0
    for i in range(10):
        a, b = iterate(a, b, x, y, 100 * (i + 1))

        prediction = model(a, b, x)
        loss = loss_function(a, b, x, y)


        if i == 0:
            diff_loss = loss
        else:
            diff_loss = abss(loss - old_loss)
            old_loss = loss




        sh.sheet1.update(('A' + str(i + 1)), str(100 * (i + 1)))
        sh.sheet1.update(('B' + str(i + 1)), str(a[0]))
        sh.sheet1.update(('C' + str(i + 1)), str(b[0]))
        sh.sheet1.update(('D' + str(i + 1)), str(diff_loss))
```
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (i > dataSet.Count)
        {
            return;
        }

        if (dataSet["iter_" + i.ToString()] > 1000 & statusStart == false & i <= dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["iter_" + i.ToString()] + " " + i.ToString());
        }

        if (dataSet["iter_" + i.ToString()] < 1000 & dataSet["iter_" + i.ToString()] > 100 & statusStart == false & i <= dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["iter_" + i.ToString()] + " " + i.ToString());
        }

        if (dataSet["iter_" + i.ToString()] < 100 & statusStart == false & i <= dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["iter_" + i.ToString()] + " " + i.ToString());
        }
    }
    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1WiQ6zudIP8BAGc32i8sB8U0b0Klndb6aI0I1X2EWYls/values/Лист1?key=AIzaSyBH3mfoAHk8DRpuESExMeNvIOMmwtrWvoo");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("iter_" + selectRow[0]), float.Parse(selectRow[2]));
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
-![image](https://user-images.githubusercontent.com/80514942/193809201-2235690e-50c6-4e95-aa79-4bbf3f0f5f20.png)



## Выводы

Научился изменять данные в таблице Google  с помощью Python, затем используя эти данные в  Unity, поработал с файлами JSON и звуками. При выводе даных из первой лабороторной работы в Google Sheets,и при дальнейшем использовании в Unity, возникли трудности с тем, что Unity ругался на саму первую итерацию, выдавал ошибку, но после того, как я исправил тем, что когда дата сет еще не подргузился мы ничего неделаем, все заработало. 



