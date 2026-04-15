<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Дашборд рынка суши в Омске</title>
  <!-- Подключаем PropTypes первым -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/prop-types/15.8.1/prop-types.min.js"></script>
  <!-- React и зависимости -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <!-- Babel для трансформации JSX -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
  <!-- PapaParse для парсинга CSV -->
  <script src="https://unpkg.com/papaparse@latest/papaparse.min.js"></script>
  <!-- Chrono для парсинга дат (не используется, но включен для совместимости) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chrono-node/1.3.11/chrono.min.js"></script>
  <!-- Tailwind CSS через CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Recharts для визуализаций -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/recharts/2.15.0/Recharts.min.js"></script>
</head>
<body class="bg-gray-100 font-sans">
  <div id="root" class="container mx-auto p-4"></div>

  <script type="text/babel">
    // Инициализация React и зависимостей
    const { useState, useEffect } = React;
    const { createRoot } = ReactDOM;
    const { BarChart, Bar, PieChart, Pie, Cell, XAxis, YAxis, Tooltip, Legend, ResponsiveContainer } = Recharts;

    // Основной компонент приложения
    const App = () => {
      // Состояние для данных и фильтров
      const [resultsData, setResultsData] = useState([]);
      const [profileData, setProfileData] = useState([]);
      const [loading, setLoading] = useState(true);
      const [ageFilter, setAgeFilter] = useState('Все');
      const [genderFilter, setGenderFilter] = useState('Все');

      // Загрузка и обработка CSV-данных
      useEffect(() => {
        // Симуляция загрузки CSV-данных
        const resultsCsv = `
Категория,Подкатегория,Количество
Цель посещения,Провести время с друзьями,177
Цель посещения,Не хочется готовить,79
Цель посещения,Попробовать что-то новое,93
Частота посещщений,1-2 раза в месяц,264
Частота посещений,Каждый день,29
Частота посещений,Чаще 3 раз в месяц,56
Известные рестораны,Японский домик,51
Известные рестораны,Суши маркет,46
Известные рестораны,Суши мастер,40
Известные рестораны,Зебры,38
Известные рестораны,Япончик,35
Самый посещаемый,Японский домик,124
Самый посещаемый,Суши маркет,46
Самый посещаемый,Япончик,37
Самый посещаемый,Суши мастер,21
Самый посещаемый,Зебры,24
Удовлетворенность,Полностью удовлетворен,161
Удовлетворенность,Скорее удовлетворен,157
Удовлетворенность,Скорее не удовлетворен,24
Удовлетворенность,Полностью не удовлетворен,5
Максимальная цена,349,43.4
Максимальная цена,249,28.6
Максимальная цена,199,14.9
Минимальная цена,139,57.1
Минимальная цена,189,15.7
Справедливая цена,199,32
Справедливая цена,189,27.4
Важность характеристик,Размер порции,24
Важность характеристик,Качество суши/роллов,22
Важность характеристик,Качество обслуживания,20
Важность характеристик,Уровень цен,18
Важность характеристик,Атмосфера,16
Оценка характеристик,Качество обслуживания,3.3
Оценка характеристик,Атмосфера,3.3
Оценка характеристик,Уровень цен,3.2
Оценка характеристик,Качество суши/роллов,3.2
Оценка характеристик,Размер порции,3.2
Любимые суши,Филадельфия,150
Любимые суши,Калифорния,80
Любимые суши,Запечённые,40
Любимые суши,Дракон,20
Любимые суши,Горячие,30
`;
        const profileCsv = `
Пол,Возрастная группа,Доход,Любимые суши,Количество
Женский,18-24,12000-19000,Филадельфия,50
Женский,18-24,20000-29000,Филадельфия,30
Мужской,18-24,12000-19000,Калифорния,20
Женский,18-24,30000-39000,Филадельфия,25
Мужской,18-24,20000-29000,Калифорния,10
Женский,25-34,20000-29000,Калифорния,15
Мужской,35-44,12000-19000,Калифорния,10
Женский,18-24,12000-19000,Запечённые,20
`;
        // Парсинг CSV с помощью PapaParse
        const parseCsv = (csv, setData) => {
          Papa.parse(csv, {
            header: true,
            skipEmptyLines: true,
            dynamicTyping: false,
            transformHeader: (header) => header.trim().replace(/^"|"$/g, ''),
            transform: (value, header) => {
              let cleaned = value.trim().replace(/^"|"$/g, '');
              if (header === 'Количество') return Number(cleaned) || 0;
              return cleaned;
            },
            complete: (results) => {
              const cleanedData = results.data.map(row => ({
                ...row,
                Количество: Number(row['Количество']) || 0
              }));
              setData(cleanedData);
            },
            error: (err) => console.error('Ошибка парсинга CSV:', err)
          });
        };

        parseCsv(resultsCsv, setResultsData);
        parseCsv(profileCsv, setProfileData);
        setLoading(false);
      }, []);

      // Фильтрация данных по выбору пользователя
      const filteredProfileData = profileData.filter(row => {
        return (ageFilter === 'Все' || row['Возрастная группа'] === ageFilter) &&
               (genderFilter === 'Все' || row['Пол'] === genderFilter);
      });

      // Подготовка данных для графиков
      const mostVisited = resultsData.filter(row => row['Категория'] === 'Самый посещаемый')
        .sort((a, b) => b['Количество'] - a['Количество'])
        .slice(0, 5);
      
      const visitPurpose = resultsData.filter(row => row['Категория'] === 'Цель посещения');
      
      const visitFrequency = resultsData.filter(row => row['Категория'] === 'Частота посещений');
      
      const satisfaction = resultsData.filter(row => row['Категория'] === 'Удовлетворенность');
      
      const maxPrice = resultsData.filter(row => row['Категория'] === 'Максимальная цена');
      const minPrice = resultsData.filter(row => row['Категория'] === 'Минимальная цена');
      const fairPrice = resultsData.filter(row => row['Категория'] === 'Справедливая цена');
      
      const importance = resultsData.filter(row => row['Категория'] === 'Важность характеристик')
        .sort((a, b) => b['Количество'] - a['Количество']);
      
      const characteristics = resultsData.filter(row => row['Категория'] === 'Оценка характеристик')
        .sort((a, b) => b['Количество'] - a['Количество']);
      
      const favoriteSushi = filteredProfileData.reduce((acc, row) => {
        const sushi = row['Любимые суши'];
        acc[sushi] = (acc[sushi] || 0) + row['Количество'];
        return acc;
      }, {});
      
      const favoriteSushiData = Object.entries(favoriteSushi).map(([name, Количество]) => ({
        name,
        Количество
      })).sort((a, b) => b.Количество - a.Количество).slice(0, 5);

      // Подготовка данных для карточек профиля потребителей
      const consumerSegments = [
        {
          title: 'Молодые женщины 18-24',
          description: 'Доход 12-19 тыс., предпочитают Филадельфию',
          count: 50,
          icon: '👩‍🦰'
        },
        {
          title: 'Мужчины 18-24',
          description: 'Доход 12-19 тыс., предпочитают Калифорнию',
          count: 20,
          icon: '👨'
        },
        {
          title: 'Женщины 25-34',
          description: 'Доход 20-29 тыс., любят Калифорнию',
          count: 15,
          icon: '👩'
        }
      ];

      // Цвета для графиков
      const COLORS = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFD700'];

      // Отображение состояния загрузки
      if (loading) {
        return <div class="text-center text-xl mt-10">Загрузка...</div>;
      }

      // Рендеринг дашборда
      return (
        <div class="bg-white p-6 rounded-lg shadow-lg">
          {/* Заголовок */}
          <h1 class="text-3xl font-bold mb-6 text-center">Дашборд рынка суши в Омске</h1>
          
          {/* Фильтры */}
          <div class="mb-6 flex flex-wrap gap-4">
            <div>
              <label class="mr-2">Возрастная группа:</label>
              <select
                class="border rounded p-2"
                value={ageFilter}
                onChange={(e) => setAgeFilter(e.target.value)}
              >
                <option value="Все">Все</option>
                <option value="18-24">18-24</option>
                <option value="25-34">25-34</option>
                <option value="35-44">35-44</option>
              </select>
            </div>
            <div>
              <label class="mr-2">Пол:</label>
              <select
                class="border rounded p-2"
                value={genderFilter}
                onChange={(e) => setGenderFilter(e.target.value)}
              >
                <option value="Все">Все</option>
                <option value="Женский">Женский</option>
                <option value="Мужской">Мужской</option>
              </select>
            </div>
          </div>

          {/* Графики */}
          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            {/* Столбчатая диаграмма: Самые посещаемые рестораны */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Топ-5 самых посещаемых суши-ресторанов</h2>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={mostVisited}>
                  <XAxis dataKey="Подкатегория" fontSize={12} />
                  <YAxis fontSize={12} />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="Количество" fill="#FF6B6B" />
                </BarChart>
              </ResponsiveContainer>
            </div>

            {/* Круговая диаграмма: Цели посещения */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Причины посещения суши-ресторанов</h2>
              <ResponsiveContainer width="100%" height={300}>
                <PieChart>
                  <Pie
                    data={visitPurpose}
                    dataKey="Количество"
                    nameKey="Подкатегория"
                    cx="50%"
                    cy="50%"
                    outerRadius={100}
                    fill="#8884d8"
                    label
                  >
                    {visitPurpose.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                    ))}
                  </Pie>
                  <Tooltip />
                  <Legend />
                </PieChart>
              </ResponsiveContainer>
            </div>

            {/* Столбчатая диаграмма: Частота посещений */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Частота посещений суши-ресторанов</h2>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={visitFrequency}>
                  <XAxis dataKey="Подкатегория" fontSize={12} />
                  <YAxis fontSize={12} />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="Количество" fill="#45B7D1" />
                </BarChart>
              </ResponsiveContainer>
            </div>

            {/* Круговая диаграмма: Удовлетворенность */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Удовлетворенность самым посещаемым рестораном</h2>
              <ResponsiveContainer width="100%" height={300}>
                <PieChart>
                  <Pie
                    data={satisfaction}
                    dataKey="Количество"
                    nameKey="Подкатегория"
                    cx="50%"
                    cy="50%"
                    outerRadius={100}
                    fill="#8884d8"
                    label
                  >
                    {satisfaction.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                    ))}
                  </Pie>
                  <Tooltip />
                  <Legend />
                </PieChart>
              </ResponsiveContainer>
            </div>

            {/* Столбчатая диаграмма: Восприятие цен */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Восприятие цен за порцию "Калифорния" (8 шт.)</h2>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={[...maxPrice, ...minPrice, ...fairPrice]}>
                  <XAxis dataKey="Подкатегория" fontSize={12} />
                  <YAxis fontSize={12} />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="Количество" fill="#96CEB4" />
                </BarChart>
              </ResponsiveContainer>
            </div>

            {/* Столбчатая диаграмма: Важность характеристик */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Важность характеристик при выборе ресторана</h2>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={importance}>
                  <XAxis dataKey="Подкатегория" fontSize={12} />
                  <YAxis fontSize={12} />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="Количество" fill="#FFD700" />
                </BarChart>
              </ResponsiveContainer>
            </div>

            {/* Столбчатая диаграмма: Оценка характеристик */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Оценка характеристик самого посещаемого ресторана</h2>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={characteristics}>
                  <XAxis dataKey="Подкатегория" fontSize={12} />
                  <YAxis fontSize={12} domain={[0, 5]} />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="Количество" fill="#4ECDC4" />
                </BarChart>
              </ResponsiveContainer>
            </div>

            {/* Столбчатая диаграмма: Любимые виды суши */}
            <div class="bg-gray-50 p-4 rounded">
              <h2 class="text-xl font-semibold mb-4">Топ-5 любимых видов суши/роллов</h2>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={favoriteSushiData}>
                  <XAxis dataKey="name" fontSize={12} />
                  <YAxis fontSize={12} />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="Количество" fill="#FF6B6B" />
                </BarChart>
              </ResponsiveContainer>
            </div>
          </div>

          {/* Профиль потребителей: Карточки */}
          <div class="mt-6">
            <h2 class="text-xl font-semibold mb-4">Ключевые сегменты потребителей</h2>
            <div class="grid grid-cols-1 md:gridcols-3 gap-4 mb-6">
              {consumerSegments.map((segment, index) => (
                <div key={index} class="bg-white p-4 rounded-lg shadow-md flex items-center">
                  <span class="text-4xl mr-4">{segment.icon}</span>
                  <div>
                    <h3 class="text-lg font-semibold">{segment.title}</h3>
                    <p class="text-gray-600">{segment.description}</p>
                    <p class="text-sm text-gray-500">Количество: {segment.count}</p>
                  </div>
                </div>
              ))}
            </div>
          </div>

          {/* Профиль потребителей: Интерактивная таблица */}
          <div class="mt-6">
            <h2 class="text-xl font-semibold mb-4">Детальный профиль потребителей</h2>
            <div class="overflow-x-auto">
              <table class="min-w-full bg-white border">
                <thead>
                  <tr>
                    <th class="py-2 px-4 border">Пол</th>
                    <th class="py-2 px-4 border">Возрастная группа</th>
                    <th class="py-2 px-4 border">Доход</th>
                    <th class="py-2 px-4 border">Любимые суши</th>
                    <th class="py-2 px-4 border">Количество</th>
                  </tr>
                </thead>
                <tbody>
                  {filteredProfileData.map((row, index) => (
                    <tr key={index}>
                      <td class="py-2 px-4 border">{row['Пол']}</td>
                      <td class="py-2 px-4 border">{row['Возрастная группа']}</td>
                      <td class="py-2 px-4 border">{row['Доход']}</td>
                      <td class="py-2 px-4 border">{row['Любимые суши']}</td>
                      <td class="py-2 px-4 border">{row['Количество']}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>

          {/* Выводы */}
          <div class="mt-6 bg-gray-50 p-4 rounded">
            <h2 class="text-xl font-semibold mb-2">Выводы</h2>
            <p>
              Рынок суши в Омске доминируют молодые потребители (18-24 года), особенно женщины, с выраженным предпочтением роллов "Филадельфия" и "Калифорния". "Японский домик" лидирует по посещаемости благодаря высокой удовлетворенности (более 80% полностью или скорее удовлетворены). Потребители ценят размер порций и качество суши, при этом справедливой ценой за порцию "Калифорния" считается 199-249 рублей. Фокус на качественные роллы "Филадельфия" для молодежи и улучшение атмосферы в ресторанах может способствовать росту рынка.
            </p>
          </div>
        </div>
      );
    };

    // Рендеринг приложения
    const root = createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
