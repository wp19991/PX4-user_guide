# Комп'ютерний зір (оптичний потік, MoCap, VIO, уникання)

Техніки [комп'ютерного зору](https://en.wikipedia.org/wiki/Computer_vision) дозволяють комп'ютерам використовувати візуальні дані для розуміння їх оточення.

PX4 використовує системи комп'ютерного зору (переважно запущені на [супутніх комп'ютерах](../companion_computer/README.md)) для підтримки наступних функцій:

- [Оптичний потік](#optical-flow) забезпечує оцінку швидкості у двох вимірах (з використанням камери, спрямованої вниз, та датчика відстані, спрямованого вниз).
- [Захоплення руху](#motion-capture) забезпечує оцінку позиції у трьох вимірах за допомогою візійної системи, яка знаходиться _поза_ транспортним засобом. Це переважно використовується для внутрішньої навігації.
- [Візуальна інерціальна оцінка положення](#visual-inertial-odometry-vio) забезпечує оцінку позиції та швидкості у трьох вимірах за допомогою вбудованої візійної системи та ІВП. Це використовується для навігації в тих випадках, коли інформація про позицію GNSS відсутня або ненадійна.
- [Уникання перешкод](../computer_vision/obstacle_avoidance.md) забезпечує повну навігацію навколо перешкод під час польоту за запланованим маршрутом (наразі підтримуються місії). Це використовує [PX4/PX4-Avoidance](https://github.com/PX4/PX4-Avoidance), яке працює на супутньому комп'ютері.
- [Уникнення зіткнення](../computer_vision/collision_prevention.md) використовується для зупинки транспортних засобів перед тим, як вони зіткнуться з перешкодою (переважно при польоті у ручних режимах).

:::tip
Набір для розвитку автономності візійної системи PX4 (від Holybro) - це надійний та доступний набір для розробників, які працюють з комп'ютерним зором на PX4.
Він поставляється без попередньо встановленого програмного забезпечення, але включає приклад реалізації уникання перешкод для демонстрації можливостей платформи.
:::

## Захоплення руху

Захоплення руху (MoCap) - це техніка оцінки тривимірного _положення_ (позиції та орієнтації) транспортного засобу з використанням механізму позиціонування, який знаходиться _поза_ транспортним засобом. Системи захоплення руху найчастіше виявляють рух за допомогою інфрачервоних камер, але також можуть використовуватися інші типи камер, лідар або ультраширокосмуговий радіорухометр (УШР).

:::note
Захоплення руху (MoCap) часто використовується для навігації транспортного засобу в ситуаціях, де відсутній GPS (наприклад, всередині приміщень) та надає позицію відносно _локальної_ системи координат.
:::

Для отримання інформації про MoCap, перегляньте:

- [Оцінка зовнішньої позиції ](../ros/external_position_estimation.md)
- [Літання з використанням захоплення руху (VICON, NOKOV, Optitrack) ](../tutorials/motion-capture.md)
- [EKF > Зовнішня візійна система ](../advanced_config/tuning_the_ecl_ekf.md#external-vision-system)

## Візуальна інерціальна оцінка положення (VIO)

Візуальна інерціальна оцінка положення (VIO) використовується для оцінки тривимірного _положення_ (позиції та орієнтації) та _швидкості_ руху транспортного засобу відносно _місцевої_ початкової позиції. Часто використовується для навігації транспортного засобу в ситуаціях, де GPS відсутній (наприклад, всередині приміщень) або ненадійний (наприклад, при польоті під мостом).

VIO використовує [візуальну одометрію](https://en.wikipedia.org/wiki/Visual_odometry) для оцінки _положення_ транспортного засобу на основі візуальної інформації, поєднаної з інерційними вимірами від ІНС (для корекції помилок, пов'язаних з швидким рухом транспортного засобу, що призводить до поганого захоплення зображення).

:::note
Однією з відмінностей між VIO і [MoCap](#motion-capture) є те, що камери/ІНС VIO базуються на транспортному засобі, і додатково надають інформацію про швидкість.
:::

Для отримання інформації щодо налаштування VIO на PX4 дивіться:

- [EKF > Зовнішня візійна система ](../advanced_config/tuning_the_ecl_ekf.md#external-vision-system)
- [Керівництво з налаштування T265 ](../peripherals/camera_t265_vio.md)

## Оптичний потік

[Оптичний потік](../sensor/optical_flow.md) забезпечує оцінку швидкості у двох вимірах (з використанням камери, спрямованої вниз, та датчика відстані, спрямованого вниз).

Для отримання інформації про оптичний потік, перегляньте:

- [Оптичний потік](../sensor/optical_flow.md)
- [EKF > Оптичний потік](../advanced_config/tuning_the_ecl_ekf.md#optical-flow)

## Порівняння

### Оптичний потік та VIO для оцінки локального положення

Обидва методи використовують камери і вимірюють різницю між кадрами. Оптичний потік використовує камеру, спрямовану вниз, тоді як VIO використовує стереокамеру або камеру з відстеженням під кутом 45 градусів. Припускаючи, що обидва методи добре калібруються, який з них краще для оцінки локального положення?

Консенсусом [здається](https://discuss.px4.io/t/vio-vs-optical-flow/34680):

Оптичний потік:

- Оптичний потік, спрямований вниз, надає вам планарну швидкість, яка коригується на кутову швидкість за допомогою гіроскопа.
- Вимагає точної відстані до землі і передбачає планарну поверхню. Враховуючи ці умови, це може бути настільки ж точним / надійним, як VIO (для польотів у приміщенні)
- Має більшу надійність, оскільки має менше станів, ніж VIO.
- Значно дешевший і легший у налаштуванні оскільки для цього потрібен лише датчик оптичного потоку, дальномір та налаштування кількох параметрів (які можна підключити до керуючого пристрою польоту).

VIO:

- Дорожче придбати і складніше налаштувати. Потребує окремий супутній комп'ютер, калібрування, програмне забезпечення, конфігурацію та інше.
- Буде менш ефективним, якщо відсутні точкові особливості для відстеження (на практиці реальний світ зазвичай має точкові особливості).
- Більш гнучкий і дозволяє додаткові функції, такі як уникання перешкод і картографування.

Комбінація (об'єднання обох) ймовірно найбільш надійна, хоча не обов'язкова в більшості реальних сценаріїв. Зазвичай ви оберете систему, яка відповідає вашому робочому середовищу, потрібним функціям та обмеженням вартості:

- Використовуйте VIO якщо ви плануєте летіти на відкритому повітрі без GPS (або на відкритому повітрі) або якщо вам потрібно підтримати уникання перешкоди та інші функції зору комп'ютера.
- Використовувати оптичний потік, якщо ви плануєте лише літати в приміщенні (без GPS) і вартість є важливою.

## Зовнішні ресурси

- [XTDrone](https://github.com/robin-shaun/XTDrone/blob/master/README.en.md) - ROS + PX4 середовище симуляцій для комп'ютерного бачення. У [Посібнику з роботи з дроном XT](https://www.yuque.com/xtdrone/manual_en) є все необхідне для початку роботи!
