# Hi there, I'm [Hyman](https://github.com/MrHyman213) ![](https://github.com/blackcater/blackcater/raw/main/images/Hi.gif) 
### Noname from noname place 🇷🇺

# Оценочный сервис отделения (commonRatesProvider). 
Данный сервис предназначен для "общения" бэкенд и фронтенд сторон.  

Основные моменты, которые  делает сервис: 
  - Обращается к гугл картам, для получения данных о длительности дороги.
      - Parser
        <pre>
    <div class="container">
        <div class="block two first">
            <h2>Your title</h2>
            <div class="wrap">
            public Container request(double lat1, double lng1, double lat2, double lng2, long idDepartment, String mode) throws JsonProcessingException {
        String body = "https://maps.googleapis.com/maps/api/distancematrix/json";
        String url = body + "?destinations=" + lat1 + "%2C" + lng1 + "&mode=" + mode + "&origins=" + lat2 + "%2C" + lng2 + "&key=" + key;
        Container container = objectMapper.readValue(jdk.performRequest(url), Container.class);
        container.setId(idDepartment);
        return container;
    }
            </div>
        </div>
    </div>
</pre>
  - Парсит данные из JSON'a.
  - Передает map-у другому сервису в формате <long-long>.
  - Выдает "рейтинг" и идентификатор лучшего отделения для фронта.
