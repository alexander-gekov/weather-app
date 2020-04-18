<template>
    <div class="text-white lg:mb-8 mb-4">
        <div class="places-input text-gray-800">
            <input type="search" id="address" class="form-control w-full" placeholder="In which city do you live?"/>

            <p>Selected: <strong id="address-value">none</strong></p>
        </div>
        <div
                class="weather-container font-sans w-full max-w-lg rounded-lg overflow-hidden bg-gray-900 shadow-lg mt-4"
        >
            <div class="current-weather flex items-center justify-between px-6 py-8">
                <div class="flex items-center">
                    <div>
                        <div class="lg:text-5xl text-2xl font-semibold">{{ currentTemperature.actual }} °C</div>
                        <div>Усеща се като {{ currentTemperature.feels }}°C</div>
                    </div>
                    <div class="pl-2">
                        <div class="font-semibold">{{ currentTemperature.summary }}</div>
                        <div class="text-lg">{{ location.name }}</div>
                    </div>
                </div>
                <div>
                    <canvas ref="iconCurrent" id="iconCurrent" width="96" height="96" class="ml-4"></canvas>
                    <div class="text-center">{{ currentTemperature.description }}</div>
                </div>
            </div>
            <div class="future-weather text-sm bg-gray-800 px-6 py-8 overflow-hidden">
                <div
                        v-for="(day,index) in dailyFiveDays"
                        :key="day.dt_text"
                        class="flex items-center"
                        :class="{'mt-8' : index > 0}"
                >
                    <div class="w-1/6 text-lg text-gray-200">{{toDayOfWeek(day.dt)}}</div>
                    <div class="w-3/6 px-4 flex items-center">
                        <div>
                            <canvas
                                    :id="`icon${index+1}`"
                                    :data-icon="getIcon(day.weather[0].icon)"
                                    width="24"
                                    height="24"
                            ></canvas>
                        </div>
                        <div class="ml-3">{{day.weather[0].description}}</div>
                    </div>
                    <div class="w-2/6 text-right">
                        <div>{{Math.round(day.main.temp_max)}} °C</div>
                        <div>{{Math.round(day.main.feels_like)}} °C</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        mounted() {

            this.fetchData();


            var placesAutocomplete = places({
                appId: 'plK25AMJHJK8',
                apiKey: 'ba6a0fb7eadec9eb0956c448c1afc42e',
                container: document.querySelector('#address'),
                templates: {
                    value: function (suggestion) {
                        return suggestion.name;
                    }
                }
            }).configure({
                lang: 'bg',
                type: 'city',
                aroundLatLngViaIP: false,
            });

            var $address = document.querySelector('#address-value');

            placesAutocomplete.on('change', (e) => {
                $address.textContent = e.suggestion.value;

                this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`;
                this.location.lat = e.suggestion.latlng.lat;
                this.location.lon = e.suggestion.latlng.lng;
            });

            placesAutocomplete.on('clear', () => {
                $address.textContent = 'none';
            });



        },
        computed: {
            dailyFiveDays() {
                return this.daily.filter((day, index) => index % 8 == 0);
            }
        },
        watch: {
          location:{
              handler(newValue,oldValue){
                  this.fetchData();
              },
              deep:true
          }
        },
        data() {
            return {
                currentTemperature: {
                    actual: "",
                    feels: "",
                    summary: "",
                    description: "",
                    icon: ""
                },
                location: {
                    name: "София, България",
                    lat: 42.7,
                    lon: 23.32
                },
                daily: []
            };
        },
        methods: {
            fetchData() {
                var skycons = new Skycons({color: "white"});

                fetch(`/api/weather?lat=${this.location.lat}&lon=${this.location.lon}`)
                    .then(response => response.json())
                    .then(data => {
                        // console.log(data);
                        this.currentTemperature.actual = Math.round(data.list[0].main.temp);
                        this.currentTemperature.feels = Math.round(
                            data.list[0].main.feels_like
                        );
                        this.currentTemperature.summary = data.list[0].weather[0].main;
                        this.currentTemperature.description =
                            data.list[0].weather[0].description;
                        this.currentTemperature.icon = this.getIcon(
                            data.list[0].weather.icon
                        );

                        this.daily = data.list;

                        console.log(this.dailyFiveDays);
                        skycons.add("iconCurrent", this.currentTemperature.icon);
                        skycons.play();

                        this.$nextTick(() => {
                            skycons.add(
                                "icon1",
                                document.getElementById("icon1").getAttribute("data-icon")
                            );
                            skycons.add(
                                "icon2",
                                document.getElementById("icon2").getAttribute("data-icon")
                            );
                            skycons.add(
                                "icon3",
                                document.getElementById("icon3").getAttribute("data-icon")
                            );
                            skycons.add(
                                "icon4",
                                document.getElementById("icon4").getAttribute("data-icon")
                            );
                            skycons.add(
                                "icon5",
                                document.getElementById("icon5").getAttribute("data-icon")
                            );
                            skycons.play();
                        });
                    });
            },
            getIcon(iconID) {
                if (iconID === "01d") {
                    return "clear-day";
                } else if (iconID === "01n") {
                    return "clear-night";
                } else if (iconID === "02d") {
                    return "partly-cloudy-day";
                } else if (iconID === "02n") {
                    return "partly-cloudy-night";
                } else if (
                    iconID === "03d" ||
                    iconID === "03n" ||
                    iconID === "04d" ||
                    iconID === "04n"
                ) {
                    return "cloudy";
                } else if (iconID === "09d" || iconID === "09n") {
                    return "rain";
                } else if (
                    iconID === "10d" ||
                    iconID === "10n" ||
                    iconID === "11d" ||
                    iconID === "11n"
                ) {
                    return "sleet";
                } else if (iconID === "13d" || iconID === "13n") {
                    return "snow";
                } else {
                    return "fog";
                }
            },
            toDayOfWeek(timestamp) {
                const newDate = new Date(timestamp * 1000);
                const days = [
                    "НЕД",
                    "ПОН",
                    "ВТО",
                    "СРЯ",
                    "ЧЕТ",
                    "ПЕТ",
                    "СЪБ"
                ];
                return days[newDate.getDay()];
            }
        }
    };
</script>
