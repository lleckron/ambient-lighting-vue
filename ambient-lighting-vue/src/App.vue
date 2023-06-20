<template>
    <button class="toggle-button" @click="toggleDivs">Swap</button>

    <div v-show="!videoShown" class="square" @click="switchImage()">
        <img class="background-img" id="background-img" :src="imgSource" @load="getImageColors('image')" alt="image" :style="shadowStyle"/>
    </div>
    
    <div v-show="videoShown" class="square">
        <video class="background-video" id="background-video" :src="vidSource" :style="shadowStyle" loop controls/>
    </div>
</template>

<script>
import banana from '@/assets/banana.webp'
import watermelon from '@/assets/watermelon.webp'
import wtfFruit from '@/assets/wtf-fruit-is-this.webp'
import ambient480p from '@/assets/ambient-480p.mp4'

export default {
    name: 'App',
    data() {
        return {
            videoShown: false,
            imgSource: banana,
			imgArray: [banana, watermelon, wtfFruit],
            imgIndex: 0,
			shadowStyle: String,
            vidSource: ambient480p,
            recentTimeUpdate: 0
        }
    },
    methods: {
        toggleDivs() {
            this.videoShown = !this.videoShown
        },

        switchImage() {
            this.imgIndex += 1
            if(this.imgIndex > 2) {
                this.imgIndex = 0
            }
            this.imgSource = this.imgArray[this.imgIndex]
        },

        getImageColors(imageFromType) {
            const image = document.getElementById('background-img')
            const video = document.getElementById('background-video')
            const canvas = document.createElement('canvas')
 
            let context
            if(imageFromType === 'image') {
                canvas.width = image.getBoundingClientRect().width
                canvas.height = image.getBoundingClientRect().height
                context = canvas.getContext('2d',  ({ willReadFrequently: true }))
                context.drawImage(image, 0, 0, canvas.width, canvas.height)
            } else if(imageFromType === 'video') {
                canvas.width = video.getBoundingClientRect().width
                canvas.height = video.getBoundingClientRect().height
                context = canvas.getContext('2d',  ({ willReadFrequently: true }))
                context.drawImage(video, 0, 0, canvas.width, canvas.height)
            } else {
                alert('There is a problem, refresh the page.')
            }

            // get pixel data from first and last row of image
            const dataTop = context.getImageData(0, 0, 1, canvas.width)
            const dataBottom = context.getImageData(canvas.height - 1, 0, 1, canvas.width)

			this.findRGBAverage(dataTop.data, dataBottom.data)
        },

		findRGBAverage(pixelDataTop, pixelDataBottom) {
			let r = 0;
			let g = 0;
			let b = 0;
			let denominator =  0;

			for(let i = 0; i < pixelDataTop.length; i += 4) {
				r += pixelDataTop[i]
				g += pixelDataTop[i + 1]
				b += pixelDataTop[i + 2]
				denominator += 1
			}

			for(let i = 0; i < pixelDataBottom.length; i += 4) {
				r += pixelDataBottom[i]
				g += pixelDataBottom[i + 1]
				b += pixelDataBottom[i + 2]
				denominator += 1
			}

			this.setAmbientLight(r, g, b, denominator)
		},

		setAmbientLight(r, g, b, denominator) {
			const avgR = Math.floor(r / denominator)
			const avgG = Math.floor(g / denominator)
			const avgB = Math.floor(b / denominator)

			const shadowOne = '0 0 10px rgba(' + avgR + ', ' + avgG + ', ' + avgB + ', 0.2)'
			const shadowTwo = '0 0 30px rgba(' + avgR + ', ' + avgG + ', ' + avgB + ', 0.4)'
			const shadowThree= '0 0 50px rgba(' + avgR + ', ' + avgG + ', ' + avgB + ', 0.6)'
			const shadowFour = '0 0 70px rgba(' + avgR + ', ' + avgG + ', ' + avgB + ', 0.8)'
			const shadowFive = '0 0 100px rgba(' + avgR + ', ' + avgG + ', ' + avgB + ', 1)'

			this.shadowStyle = 'box-shadow: ' + shadowOne + ', ' + shadowTwo + ', ' + shadowThree + ', ' + shadowFour + ', ' + shadowFive
		},

        setupVideoEventListeners() {
            const video = document.getElementById('background-video')
            video.addEventListener('timeupdate', () => this.getImageColors('video'))
        }
    },
    mounted() {
        this.setupVideoEventListeners()
    }
}
</script>

<style>
html, body {
    padding: 0;
    margin: 0;
    width: 100vw;
    height: 100vh;
	background: #121212;
}

#app {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
}

.square {
    display: flex;
    position: relative;
    width: 75%;
    height: 75%;
}

.square:hover {
    cursor: pointer;
}

.background-img {
    height: 100%;
    width: 100%;
    background-size: cover;
}

.background-video {
    height: 100%;
    width: 100%;
    transition: box-shadow .35s;
}

.toggle-button {
    position: absolute;
    top: 0;
    left: 0;
    background: transparent;
    border: 2px solid #e9e9e9;
    border-radius: 6px;
    font-size: 25px;
    color: #e9e9e9;
    margin: 30px 0 0 30px;
}

.toggle-button:hover {
    cursor: pointer;
    color: #c9c9c9;
    border-color: #c9c9c9;
}
</style>