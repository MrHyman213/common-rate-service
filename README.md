# Hi there, I'm [Hyman](https://github.com/MrHyman213) ![](https://github.com/blackcater/blackcater/raw/main/images/Hi.gif) 
### Noname from noname place 🇷🇺

# Оценочный сервис отделения (commonRatesProvider). 
Данный сервис предназначен для "общения" бэкенд и фронтенд сторон.  

Основные моменты, которые  делает сервис: 
  - Обращается к гугл картам, для получения данных о длительности дороги.
      - `Parser`
        ```
        public Container request(double lat1, double lng1, double lat2, double lng2, long idDepartment, String mode) throws JsonProcessingException {
                ...
        }
        ```
      - DTOs используемые для парсинга JSON'a.
          - `Container`
        ```
        public class Container {
            @JsonCreator
            public Container(@JsonProperty("destination_addresses") List<String> destination,
                                @JsonProperty("origin_addresses") List<String> origin,
                             @JsonProperty("rows") List<Rows> rows,
                             @JsonProperty String status) {
                  ...
            }
        }
        ```
          - `Rows`
        ```
        public class Rows {
            private List<Elements> elements;
        }
        ```
        - `Elements`
        ```
        public class Elements {
            private Distance distance;
            private Duration duration;
            private String status;
        }  
        ```
        - `Distance`
        ```
        public class Distance {
            private String text;
            private int value;
        }
        ```
        - `Duration`
        ```
        public class Duration {
            private String text;
            private int value;
        }
        ```
  - Передает map'у другому сервису в формате <long-long>.
    - `TimePathService`
      ```
      public Map<Long, Long> getRatesByRoads(InfoToGetRates infoToGetRates) throws JsonProcessingException {
            ...
      }
      ```
    Где первый long - id отделения, а второй - длительность пути в милисекундах.
    - Класс, который принимает метод getRatesByRoads()
      - `InfoToGetRates`
      ```
        public class InfoToGetRates {
            @JsonCreator
            public InfoToGetRates(@JsonProperty("cords") List<Coords> coords,
                               @JsonProperty("id") List<Integer> id,
                               @JsonProperty("user_geo") Coords userGeo,
                               @JsonProperty("move_type") String moveType,
                               @JsonProperty("service_type") String serviceType){
                  ...
            }
        }
      ```
  - Выдает "рейтинг" и идентификатор лучшего отделения для фронта в контроллере.
    - `CommonRateController`
      ```
      public ResponseEntity<?> getCommonRates(@RequestBody InfoToGetRates infoToGetRates) {
                ...
            }
      ```
  - При обращении к end-point'у "/offices/optimal", будет возвращаться JSON формата
      ```
        {
          "id_Отделения":"рейтинг"
        }
      ```
