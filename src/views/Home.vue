<template>
    <div class="home" v-if="date < 1531065600000">
        <div class="tips" :class="{'show' : isShowTips}"><img src="./img/bt.svg" alt="" class="tips-img">长按保存图片</div>
        <img id="final" :src="dataUrl" alt="" v-if="isShowCanvas">

        <div class="input-box" v-if="!isShowCanvas">
            <div class="space"></div>
            <div class="des">完善信息，生成专属海报，一起为
                Lemo上线交易所倒计时</div>
            <div class="space"></div>
            <div class="input-group"><span>名字：</span><i-input type="text" class="i-input" v-model="name" maxlength="20"></i-input></div>
            <div class="input-group"><span>职位：</span><i-input type="text" class="i-input" v-model="position" maxlength="20"></i-input></div>
            <div class="input-group"><span>头像：</span><input type="file" id="file"></div>
            <button class="btn" @click="ok">确定</button>
        </div>
        <img :src="result" alt="" id="img">
        <canvas id="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
        <img :src="leftDaysUrl" id="bg">
        <img src="./img/qr.png" id="qr" alt="">
    </div>
</template>

<script>
    import {Indicator, Toast} from 'mint-ui'

    export default {
        name: 'home',
        data() {
            return {
                result: '',
                a: 1,
                canvasWidth: window.innerWidth * window.devicePixelRatio,
                canvasHeight: window.innerHeight * window.devicePixelRatio,
                dataUrl: '',
                date: new Date().getTime(),
                position: '',
                name: '',
                isShowCanvas: false,
                isShowTips: false
            }
        },
        computed: {
            leftDays() {
                return Math.floor((1531065600000 - this.date) / 86400000)
            },
            leftDaysUrl() {
                const day = this.leftDays > 3 || this.leftDays < 0 ? 3 : this.leftDays
                return require(`./img/bg_${day}.png`)
            }
        },
        methods: {
            async readFile(e) {
                let file = e.target.files[0]
                if (!/image\/\w+/.test(file.type)) {
                    Toast({
                        message: '请确保文件为图像类型',
                        duration: 1500
                    })
                    return
                }
                let reader = new FileReader()
                reader.readAsDataURL(file)
                let url = ''
                reader.onload = function(e) {
                    url = e.target.result
                }
                await setTimeout(() => {
                    this.result = url
                }, 300)
            },
            draw() {
                setTimeout(() => {
                    this.init()
                    const scale = window.devicePixelRatio
                    const bg = document.getElementById('bg')
                    const img = document.getElementById('img')
                    const qr = document.getElementById('qr')
                    const bgRate = bg.offsetHeight / bg.offsetWidth
                    const imgRate = this.canvasWidth / 375
                    const canvas = document.getElementById('canvas')
                    let ctx = canvas.getContext('2d')
                    ctx.fillStyle = '#110735'
                    ctx.fillRect(0, 0, this.canvasWidth, this.canvasHeight)
                    ctx.drawImage(bg, 0, 0, this.canvasWidth, this.canvasWidth * bgRate)
                    ctx.drawImage(qr, 231 * imgRate, 441 * imgRate, 93 * scale, 93 * scale)
                    let grd=ctx.createLinearGradient(0,0,200,0)
                    grd.addColorStop(0, '#09E0C6')
                    grd.addColorStop(1, '#4CB4EB')
                    ctx.fillStyle = grd
                    ctx.roundRect(60 * imgRate + 60 * scale - 40, 375 * imgRate + 12 * scale, 220 * imgRate, 46 * scale, 100)
                    ctx.fill()
                    ctx.font = `${13 * scale}px Arial`;
                    ctx.fillStyle = 'black'
                    ctx.fillText(`我是${this.name}`, 60 * imgRate + 80 * scale, 375 * imgRate + 30 * scale)
                    ctx.fillText(`${this.position}`, 60 * imgRate + 80 * scale, 375 * imgRate + 50 * scale)
                    ctx.save() // 保存当前ctx的状态
                    ctx.lineWidth = 5 * scale;
                    ctx.strokeStyle = '#4CB4EB'
                    ctx.beginPath()
                    ctx.arc(60 * imgRate + 35 * scale, 375 * imgRate + 35 * scale, 35 * scale, 0, 2*Math.PI) //画出圆
                    ctx.closePath()
                    ctx.stroke()
                    ctx.clip() //裁剪上面的圆形在刚刚裁剪的园上画图
                    ctx.drawImage(img, 60 * imgRate, 375 * imgRate, 70 * scale, 70 * scale)
                    ctx.restore() // 还原状态
                    this.dataUrl = canvas.toDataURL('image/jpeg')
                    this.isShowCanvas = true
                    this.isShowTips = true
                    Indicator.close()
                }, 1000)
            },
            init() {
                CanvasRenderingContext2D.prototype.roundRect = function (x, y, w, h, r) {
                    var min_size = Math.min(w, h);
                    if (r > min_size / 2) r = min_size / 2;
                    // 开始绘制
                    this.beginPath();
                    this.moveTo(x + r, y);
                    this.arcTo(x + w, y, x + w, y + h, r);
                    this.arcTo(x + w, y + h, x, y + h, r);
                    this.arcTo(x, y + h, x, y, r);
                    this.arcTo(x, y, x + w, y, r);
                    this.closePath();
                    return this;
                }
            },
            ok() {
                if (!this.name) {
                    Toast({
                        message: '请填写姓名',
                        duration: 1500
                    })
                    return
                }
                if (!this.position) {
                    Toast({
                        message: '请填写职位',
                        duration: 1500
                    })
                    return
                }
                if (!this.result) {
                    Toast({
                        message: '请上传照片',
                        duration: 1500
                    })
                    return
                }
                Indicator.open()
                this.draw()
            },
        },
        mounted() {
            let input = document.getElementById('file')
            if (typeof FileReader === 'undefined') {
                input.setAttribute('disabled','disabled')
            } else {
                input.addEventListener('change', this.readFile, false)
            }
        }
    }
</script>
<style rel="stylesheet/scss" lang="scss">
    .home {
        $percent: 75%;
        $subpercent: 80%;
        height: 100vh;
        width: 100vw;
        overflow: hidden;
        background: rgb(17,7,53);
        text-align: center;
        .tips {
            position: fixed;
            top: 181px;
            right: -125px;
            font-size: 14px;
            z-index: 1;
            height: 26px;
            color: white;
            transition: 0.4s all;
            line-height: 26px;
            width: 125px;
            background-image: linear-gradient(53deg, #09E0C6 0%, #4CB4EB 100%);
            border-radius: 13px 0 0 13px;
            .tips-img {
                display: inline-block;
                vertical-align: middle;
                position: relative;
                top: -1px;

            }
        }
        .show {
            right: -5px;
        }
        .des {
            font-size: 16px;
            color: #05E3C4;
            letter-spacing: 0;
            text-align: center;
            width: 306px;
            margin: 60px auto 0;
        }
        .space {
            height: 1px;
        }
        #final {
            width: 100vw;
        }
        #img {
            opacity: 0;
        }
        .input-group {
            justify-content: space-between;
            display: flex;
            align-items: center;
            margin: 40px auto;
            width: $percent;
            &:nth-of-type(4) {
                margin-top: 70px;
            }
        }
        .ivu-input {
            height: 45px;
            background: rgba(5,227,196,0.22);
            border: 1px solid #05E3C4;
            color: #05E3C4;
            font-size: 14px;
        }
        .i-input {
            display: inline-block;
            width: $subpercent;
            height: 45px;
        }
        .btn {
            background: #05E3C4;
            border: 1px solid #05E3C4;
            border-radius: 5px;
            height: 45px;
            width: $percent;
            font-size: 16px;
        }
        #file {
            /*height: 45px;*/
            font-size: 16px;
            /*background: #05E3C4;*/
            /*border: 1px solid #05E3C4;*/
            border-radius: 5px;
            vertical-align: middle;
            line-height: 30px;
            color: #05E3C4;
            width: $subpercent;
        }
        .input-box {
            min-height: 100vh;
            text-align: center;
            background: url('./img/input_bg.png') no-repeat 0 0/contain;
            span {
                font-size: 14px;
                color: #05E3C4;
            }
        }
    }
</style>