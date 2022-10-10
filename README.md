# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
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
познакомиться с программными средствами для создания
системы машинного обучения и ее интеграции в Unity.

## Задание 1
- Реализовать систему машинного обучения в связке Python -
Google-Sheets – Unity.


- Создаем проект в Unity, добовляем в папку с проектом json файлы



-Открываем Anaconda Promt и пишем команды для создания и активации нового ML агента



![image](https://user-images.githubusercontent.com/80514942/194835821-84983737-1208-4c31-aeb3-0af7b3fc0cfb.png)


![image](https://user-images.githubusercontent.com/80514942/194835961-461e020e-7cf4-4bc5-9528-5f6282c8c110.png)


![image](https://user-images.githubusercontent.com/80514942/194835984-df99956b-b8ed-4adc-864a-5b4817d58dd2.png)


- Далее в консоли переходим в папку с проектом

- Создаем объекты плоскости, куба и сферы, выставляем настройки у компонентов, добавляем материалы, к

![image](https://user-images.githubusercontent.com/80514942/194835438-752ed1bd-e747-40e0-8b5f-377199bf81c0.png)


 ![image](https://user-images.githubusercontent.com/80514942/194836164-75f90456-cd1c-4a71-91b2-22c2beb3778b.png)


- В инспекторе добавялем в нашу сферу RigidBody, C# скрипт и тд


 ![image](https://user-images.githubusercontent.com/80514942/194836685-faeb1c03-2190-414a-b805-c6b8bdc45e89.png)



```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8 - 4, 0.5f, Random.value * 8 - 4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if (distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}


```

- Добавялем в корень проекта файл конфигурации нейронной сети, запускаем работу ml агента
- После прогона делаем 3, 9, 27 копий модели «Плоскость-Сфера-Куб», запускаем
симуляцию сцены и наблюдаем за результатом обучения модели.

![image](https://user-images.githubusercontent.com/80514942/194872880-a0e5957f-1ffe-4cf6-af49-d9f70065c202.png)



## Задание 2
```yaml
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
```

-trainer_type: ppo - Тип тренера ppo или sac



 batch_size - Количество опытов на каждой итерации градиентного спуска. Это всегда должно быть в несколько раз меньше, чем buffer_size. Если вы используете непрерывное пространство действий, это значение должно быть большим (порядка 1000). Если вы используете дискретное пространство действий, это значение должно быть меньше (порядка 10 секунд).
 
 
 
 buffer_size - Количество опыта, который необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы приступим к какому-либо изучению или обновлению модели. Это должно быть в несколько раз больше, чем batch_size. Обычно больший размер буфера соответствует более стабильным обновлениям обучения.
 
 
 
  learning_rate - Начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. Обычно это значение следует уменьшить, если тренировка нестабильна, а вознаграждение не увеличивается последовательно.
 
 
 
 beta-Сила регуляризации энтропии, которая делает политику "более случайной". Это гарантирует, что агенты должным образом исследуют пространство действий во время обучения. Увеличение этого значения обеспечит выполнение большего количества случайных действий. Это должно быть скорректировано таким образом, чтобы энтропия  медленно уменьшалась вместе с увеличением вознаграждения. Если энтропия падает слишком быстро, нужно увеличить бета-версию. Если энтропия падает слишком медленно, уменьшить бета.
  
  
  
  
  epsilon- Влияет на то, насколько быстро политика может развиваться во время обучения. Соответствует допустимому порогу расхождения между старой и новой политиками при обновлении с градиентным спуском. Установка этого значения небольшим приведет к более стабильным обновлениям, но также замедлит процесс обучения.
  
  
  lambd- Параметр регуляризации (лямбда), используемый при расчете Обобщенной оценки преимущества (GAE). Это можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при расчете обновленной оценки стоимости. Низкие значения соответствуют большей зависимости от текущей оценки ценности (что может быть большой погрешностью), а высокие значения соответствуют большей зависимости от фактических вознаграждений, полученных в окружающей среде (что может быть высокой дисперсией). Параметр обеспечивает компромисс между ними, и правильное значение может привести к более стабильному процессу обучения.
  
  num_epoch- Количество проходов, которые необходимо выполнить через буфер опыта при выполнении оптимизации градиентного спуска.Чем больше размер пакета, тем больше это допустимо сделать. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения.
  
  
  learning_rate_schedule-  Определяет, как скорость обучения меняется с течением времени. 
  
  
  
  normalize- Применяется ли нормализация к входным данным векторного наблюдения. Эта нормализация основана на текущем среднем значении и дисперсии векторного наблюдения. Нормализация может быть полезна в случаях со сложными задачами непрерывного управления, но может быть вредна при более простых задачах дискретного управления.
  
  
  hidden_units- Количество единиц в скрытых слоях нейронной сети. Соответствует количеству единиц в каждом полностью подключенном слое нейронной сети. Для простых задач, где правильным действием является простая комбинация входных данных наблюдения, это должно быть небольшим. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, это должно быть больше.
  
  
  num_layers- Количество скрытых слоев в нейронной сети. Соответствует количеству скрытых слоев, присутствующих после ввода наблюдения или после кодирования CNN визуального наблюдения. Для простых задач меньшее количество слоев, скорее всего, будет обучаться быстрее и эффективнее. Для более сложных задач управления может потребоваться больше уровней.
  
  gamma- Коэффициент дисконтирования для будущих вознаграждений, поступающих от окружающей среды. Это можно рассматривать как то, насколько далеко в будущем агент должен заботиться о возможных вознаграждениях. В ситуациях, когда агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в отдаленном будущем, это значение должно быть большим. В тех случаях, когда вознаграждение более немедленное, оно может быть меньше. Должно быть строго меньше 1.
  
   strength- Фактор, на который можно умножить вознаграждение, получаемое от окружающей среды. Типичные диапазоны будут варьироваться в зависимости от сигнала вознаграждения.
   
   
   max_steps - максимальное количество проходов
   
   time_horizon - временные рамки 
   
   
   summary_freq- суммированная частота 
   
   
   
   Decision Requester - запрос на принятие решения вызывает CollectObservation, а затем получает последнее действие в OnActionReceived, основанное на этом новом собранном наблюдении. С действиями из TakeActionBetweenDecisions он только снова вызовет OnActionReceived без сбора новых наблюдений и выведет последнее действие, которое он получил от NN.
   
   
  Behavior Parameters- Параметры поведения — У каждого Агента должно быть определенное поведение. Поведение определяет, как Агент принимает решения. Максимальный шаг — Определяет, сколько шагов моделирования может произойти до окончания эпизода Агента.
  
  

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



