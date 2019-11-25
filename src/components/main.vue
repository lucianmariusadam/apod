<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>
      <span class="button" @click="prev()">&lt;</span>
      <span>{{ simpleDate() }}</span>
      <span class="button" @click="next()" v-show="allowNext()">&gt;</span>
    </h2>
    <h4 id="status">{{ status }}</h4>
    <div v-show="loading" id="loading">{{ loadingMsg }}</div>
    <div v-show="!loading">
    </div>
    <div v-show="!loading">
      <img v-show="imageUrl" :src="imageUrl" :width="width" :height="height" @load="loadingOff()">
      <iframe v-show="videoUrl" :src="videoUrl" :width="width" :height="height" @load="loadingOff()"></iframe>
      <div id="explanation">{{ explanation }}</div>
    </div>
  </div>
</template>

<script>
    import axios from 'axios';
    var src='https://api.nasa.gov/planetary/apod?api_key=LhxNNyeOUIv4VadRAeUUcCFVNyFXnJNZArX8fn9Z&date=';

    export default {
        name: 'Main',
        data() {
            return {
                msg: 'NASA - Astronomy Picture of the Day',
                loadingMsg: 'loading ...',
                status: '',
                cache: {},
                media: null,
                date: null,
                maxDate: null,
                width: 320,
                height: 240,
                loading: false
            }
        },
        mounted() {
            this.init();
        },
        computed: {
            mediaDate() {
                return this.media ? this.media.date : '';
            },
            imageUrl() {
                return (this.media && this.media.media_type === 'image') ? this.media.url : '';
            },
            videoUrl() {
                return (this.media && this.media.media_type === 'video') ? this.media.url : '';
            },
            explanation() {
                return this.media ? this.media.explanation : '';
            }
        },
        methods: {
            init() {
                this.maxDate = new Date();
                this.setDate();
            },
            setDate(date) {
                if(typeof date === 'undefined') {
                    this.date = new Date();
                } else {
                    this.date.setDate(date);
                }

                this.getImage();
            },
            prev() {
                this.setDate(this.date.getDate() - 1);
            },
            next() {
                if(!this.allowNext()) {
                    return;
                }
                this.setDate(this.date.getDate() + 1);
            },
            allowNext() {
                return this.date < this.maxDate;
            },
            getImage() {
                const self = this;
                let date = this.simpleDate();

                this.media = null;

                let cached = localStorage.getItem(date);

                if(cached) {
                    self.getMediaFromCache();
                } else {
                    const url = src + date;
                    self.loadingOn();
                    axios.get(url)
                        .then(function (response) {
                            let data = response.data;
                            localStorage.setItem(date, JSON.stringify(data));
                            self.getMediaFromCache();
                        })
                        .catch(function (error) {
                            self.status = error.response.data.msg;
                            self.loadingOff();
                        })
                        .finally(function () {
                        });
                }
            },
            getMediaFromCache() {
                this.media = JSON.parse(localStorage.getItem(this.simpleDate()));
                this.status = '';
            },
            loadingOn() {
                this.loading = true;
            },
            loadingOff() {
                this.loading = false;
            },
            simpleDate() {
                if (!this.date) {
                    return null;
                }
                var d = this.date.getDate();
                var m = this.date.getMonth() + 1;
                var y = this.date.getFullYear();
                return `${y}-${m}-${d}`;
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }

  #status {
    color: red;
  }

  .button {
    font-weight: bold;
    padding: 5px;
    margin: 10px;
    cursor: pointer;
  }

  #loading {
    color: green;
  }
</style>
