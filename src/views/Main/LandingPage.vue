<template lang="pug">
section#section.intro
    .container 
        img.mainImg(src="@/assets/img/icons/main illu.svg")
        span Skapi BETA 1.0
        h1 The Future of #[br] Development
        p Frontend Driven Backend Service
        br
        a(href='https://docs.skapi.com' target="_blank")
            img.btnImg(src="@/assets/img/icons/Asset6.svg")
            span Get Started

section#section.video
    .container 
        .desc
            h4 Serverless sdasdasd
            p A robust and easy-to-use API service #[br] designed #[span to help developers build faster, #[br] more efficient, and scalable applications.] 
            a(href='https://docs.skapi.com' target="_blank")
                sui-button Getting Started
        .youtube 
            iframe.video(src="https://www.youtube.com/embed/c5jdIE9wl_8?&rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen)

section#section.features 
    .container 
        h5 Key Features
        h4 All the APIs for building incredible products
        .cardWrap 
            .card 
                .icon
                    img(src="@/assets/img/icons/Serverless.svg")
                h4.tit Serverless
                p.desc Embrace the simplicity of skapi's 100% serverless architecture. Say goodbye to dealing with terminals, installing frameworks on your machine,
            .card 
                .icon
                    img(src="@/assets/img/icons/Database.png")
                h4.tit Database
                p.desc Experience the power of an auto-indexed, scalable database that combines the best of relational and NoSQL architectures. skapi's innovative database.
            .card 
                .icon
                    img(src="@/assets/img/icons/Authentication.png")
                h4.tit Authentication
                p.desc Simplify your authentication process with skapi. From basic user login features to complex verifications, our platform offers the e
            .card 
                .icon
                    img(src="@/assets/img/icons/landing_page_html_logo.png")
                h4.tit HTML Friendly
                p.desc Take advantage of unlimited, pay-as-you-go cloud storage with skapi. Whether it's web hosting or serving large files with securi

section#section.start(ref="startSection")
    .container 
        h5 SDKS
        h4 Quick to Get Started, Easy to Grow
        .startWrap
            .steps
                .bar(ref="stepBar" :class="{ active: activeBar }")
                .cont 
                    .step 
                        .tit Create Service
                        .desc Login and Create new Service from your dashboard.
                    .step 
                        .tit Retrieve ID
                        .desc Retrieve the service ID and the owner’s ID from the dashboard.
                    .step 
                        .tit Initialize Skapi
                        .desc Import and Initialize Skapi from index.html. Replace #[em ‘SERVICE_ID’] and #[em ‘OWNERS_ID’] with appropriate value.
            .code
                .codeCopy(@click="codeCopy")
                    img(src="@/assets/img/icons/copy.svg")
                .codeNum 1#[br]2#[br]3#[br]4#[br]5#[br]6#[br]7#[br]8#[br]9
                .preCode
                    pre
                        code
                            |&lt;!DOCTYPE html&gt;
                            |&lt;head&gt;
                            |&lt;script src="https://cdn.jsdelivr.net/npm/skapi-js@latest/dist/skapi.js"&gt;&lt;/script&gt;
                            |&lt;/head&gt;
                            |&lt;script&gt;
                            |    let skapi = new Skapi('
                            span SERVICE_ID
                            | ', '
                            span OWNERS_ID
                            | ');
                            |&lt;/script&gt;

section#section.signup 
    .cont
        p 
            | Skapi is currently in BETA. We're inviting you to join us in shaping the future of web development. 
            br
            | As a BETA user, you'll enjoy full access to our services, absolutely free. 
            br
            | Join and help us create powerful software that empowers web developers around the world.
        router-link(to="/signup")
            button Sign-up
</template>

<script setup>
import { onMounted, ref } from 'vue';

let stepBar = ref(null);
let startSection = ref(null);
let activeBar = ref(false);

onMounted(() => {
    document.querySelector('main').classList.add('landing');

    window.addEventListener('scroll', () => {
    let currentScroll = window.scrollY + (window.innerHeight / 1.5);

    if(currentScroll >= startSection.value.offsetTop) {
        activeBar.value = true;
    } else {
        activeBar.value = false;
    }
})
});
</script>

<style lang="less">
main {
    &.landing {
        height: 100%;
        overflow-x: hidden;
        background-color: #f5f5f5;
        color: #262626;
    }

    #section {
        .container {
            margin: 0 auto;
            width: 1280px;

            h1 {
                font-size: 80px;
                margin: 40px 0;
            }
            h4 {
                font-size: 36px;
                font-weight: 700;
                margin: 28px 0 100px 0;
            }
            h5 {
                font-size: 28px;
                font-weight: 700;
                color: #293FE6;
                margin: 0;
            }
        }

        &.intro {
            height: 740px;
            margin: 75px;
            font-weight: 700;

            .container {
                position: relative;
                width: 1280px;
                height: 100%;

                .mainImg {
                    position: absolute;
                    right: -80px;
                    bottom: 0px;
                    width: 1280px;
                }
                span {
                    font-size: 24px;
                }
                p {
                    display: inline-block;
                    font-size: 32px;
                    font-weight: 500;
                    margin: 0 0 50px 0;
                    background: linear-gradient(180deg, #FAFF00 0%, rgba(255, 230, 0, 0.00) 97.55%);
                }
                a {
                    position: relative;
                    text-align: right;
                    text-decoration: none;
                    font-size: 24px;
                    color: #fff;

                    .btnImg {
                        position: absolute;
                        left: 0;
                        top: -10px;
                        // transform: translate(-50%, -50%);
                        transform: rotate(180deg);
                        width: 290px;
                    }

                    span {
                        position: absolute;
                        left: 90px;
                        z-index: 2;
                        white-space: nowrap;
                    }
                }
            }
        }

        &.video {
            .container {
                display: flex;
                flex-wrap: nowrap;
    
                .desc {
                    width: 50%;
                    h4 {
                        margin: 40px 0;
                    }
                    p {
                        font-size: 24px;
                        font-weight: 500;
                        line-height: 32px;
                        margin: 0 0 40px 0;
        
                        span {
                            color: #293FE6;
                        }
                    }
                }
                .youtube {
                    width: 50%;

                    .video {
                        width: 100%;
                        height: 100%;
                        border-radius: 8px;
                        box-shadow: 7px 7px 18px #bbb;
                    }
                }
            }
        }

        &.features {
            margin: 200px 0;

            .cardWrap {
                width: 100%;
                display: flex;
                flex-wrap: nowrap;

                &:last-child {
                    margin-right: 0;
                }
                
                .card {
                    &:first-child {
                        .icon {
                            padding: 7px 0;
                        }
                    }
                    &:last-child {
                        .icon {
                            padding: 10px 0;
                        }
                    }
                    width: 23%;
                    margin-right: 2%;

                    .icon {
                        height: 70px;
                        img {
                            height: 100%;
                        }
                    }
                    .tit {
                        font-size: 24px;
                        font-weight: 700;
                        margin: 27px 0;
                    }
                    .desc {
                        font-size: 20px;
                        font-weight: 400;
                        line-height: 28px;
                    }
                }
            }
        }

        &.start {
            margin-bottom: 200px;

            .startWrap {
                width: 100%;
                display: flex;
                flex-wrap: nowrap;

                .steps {
                    width: 33%;
                    margin-right: 2%;
                    display: flex;
                    flex-wrap: nowrap;

                    .bar {
                        width: 30px;
                        height: 100%;
                        background-color: #ddd;
                        margin-right: 20px;

                        &.active {
                            background-image: linear-gradient(to bottom, #001CE9 0%, #fff 96%);
                            animation: fadeInText 1s ease-out forwards;
                        }

                        @keyframes fadeInText {
                            0% {
                                opacity: 0;
                            }

                            100% {
                                opacity: 1;
                            }
                        }
                    }
                    .cont {
                        .step {
                            margin-bottom: 40px;
        
                            &:last-child {
                                margin-bottom: 0;
                            }
                            .tit {
                                font-size: 24px;
                                font-weight: 700;
                                margin-bottom: 12px;
                            }
                            .desc {
                                font-size: 16px;
                                font-weight: 400;
                                line-height: 24px;

                                em {
                                    color: #293FE6;
                                }
                            }
                        }
                    }
                }
                .code {
                    position: relative;
                    width: 65%;

                    .preCode {
                        position: relative;
                        height: 100%;
                        background-color: #434343;
                        border-radius: 8px;
                        color: #fff;
                        overflow-x: auto;
                        font-size: 16px;
                        font-weight: 400;
                        line-height: 24px;
                        display: flex;
                        // align-items: center;
                        padding: 40px 60px 60px 65px;

                        span {
                            color: #7C8CFF;
                        }
                    }

                    .codeNum {
                        position: absolute;
                        top: 55px;
                        left: 30px;
                        width: 100%;
                        height: 100%;
                        z-index: 2;
                        color: #666;
                        line-height: 25px;
                    }

                    .codeCopy {
                        position: absolute;
                        right: 24px;
                        top: 24px;
                        width: 24px;
                        height: 24px;
                        z-index: 99;

                        img {
                            width: 24px;
                            height: 24px;
                            filter: invert(100%);
                            cursor: pointer;
                        }
                    }
                }
            }
        }

        &.signup {
            width: 100%;
            height: 375px;
            background: url("src/assets/img/icons/Asset 33.png") no-repeat;
            background-size: cover;
            display: flex;
            align-items: center;
            justify-content: center;
            
            .cont {
                text-align: center;
                
                p {
                    font-size: 20px;
                    font-weight: 500;
                    color: #fff;
                    margin: 0 0 30px 0;
                }
                button {
                    position: relative;
                    width: 180px;
                    height: 40px;
                    font-weight: 700;
                    font-size: 16px;
                    color: #fff;
                    background: rgba(217, 217, 217, 0.08);
                    border: 1px solid rgba(255, 255, 255, 0.65);
                    border-radius: 4px;
                    cursor: pointer;
                }
            }

        }
    }
}
</style>