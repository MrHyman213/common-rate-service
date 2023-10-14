# Hi there, I'm [Hyman](https://github.com/MrHyman213) ![](https://github.com/blackcater/blackcater/raw/main/images/Hi.gif) 
### Noname from noname place 🇷🇺

# Оценочный сервис отделения (commonRatesProvider). 
Данный сервис предназначен для "общения" бэкенд и фронтенд сторон.  

Основные моменты, которые  делает сервис: 
  - Обращается к гугл картам, для получения данных о длительности дороги.
      - `Parser`
        ```
        public Container request(double lat1, double lng1, double lat2, double lng2, long idDepartment, String mode) throws JsonProcessingException {
                String body = "https://maps.googleapis.com/maps/api/distancematrix/json";
                String url = body + "?destinations=" + lat1 + "%2C" + lng1 + "&mode=" + mode + "&origins=" + lat2 + "%2C" + lng2 + "&key=" + key;
                Container container = objectMapper.readValue(jdk.performRequest(url), Container.class);
                container.setId(idDepartment);
                return container;
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
                destinationAddresses = destination;
                originAddresses = origin;
                this.rows = rows;
                this.status = status;
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
          times = new HashMap<>();
          for (int i = 0; i < infoToGetRates.getId().size(); i++){
              Container container = parser.request(
                      infoToGetRates.getCoords().get(i).getLat(),
                      infoToGetRates.getCoords().get(i).getLng(),
                      infoToGetRates.getUserGeo().getLat(),
                      infoToGetRates.getUserGeo().getLng(),
                      infoToGetRates.getId().get(i),
                      infoToGetRates.getMoveType()
              );
              times.put(Long.valueOf(infoToGetRates.getId().get(i)),
                      Long.valueOf(container.getRows().get(0)
                              .getElements().get(0)
                              .getDuration().getValue())
              );
          }
         return times;
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
            this.coords = coords;
            this.id = id;
            this.userGeo = userGeo;
            this.moveType = moveType;
            this.serviceType = serviceType;
        }
    }
      ```
  - Выдает "рейтинг" и идентификатор лучшего отделения для фронта в контроллере.
    - `CommonRateController`
      ```
      public ResponseEntity<?> getCommonRates(@RequestBody InfoToGetRates infoToGetRates){
        log.info("getCommonRates officesIds:{}", infoToGetRates.getId());
        Map<Long, Double> commonRatesMap;
        try {
            Map<Long, Long> idsToTime = timePathService.getRatesByRoads(infoToGetRates);
            commonRatesMap = formCommonRateService.getCommonRatesMap(idsToTime, infoToGetRates.getServiceType());
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
        return ResponseEntity.ok(commonRatesMap);
      }
      ```
      При обращении к end-point'у "/offices/optimal", будет возвращаться JSON формата
  ```
{
  "id_Отделения":"рейтинг"
}
  ```
