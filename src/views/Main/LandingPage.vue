<template lang="pug">
section#section.intro
    .container 
        img.mainImg(src="@/assets/img/icons/main illu.svg")
        span Skapi BETA 1.0
        h1 The Future of #[br] Development
        p Frontend Driven Backend Service
        br
        //- a(href='https://docs.skapi.com' target="_blank")
        //-     img(src="@/assets/img/icons/Asset6.svg")
        //-         span Getting Started 

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
                .right
                    h4.tit Serverless
                    p.cont Embrace the simplicity of skapi's 100% serverless architecture. Say goodbye to dealing with terminals, installing frameworks on your machine,
            .card 
                .icon
                    img(src="@/assets/img/icons/Database.png")
                .right
                    h4.tit Database
                    p.cont Experience the power of an auto-indexed, scalable database that combines the best of relational and NoSQL architectures. skapi's innovative database.
            .card 
                .icon
                    img(src="@/assets/img/icons/Authentication.png")
                .right
                    h4.tit Authentication
                    p.cont Simplify your authentication process with skapi. From basic user login features to complex verifications, our platform offers the e
            .card 
                .icon
                    img(src="@/assets/img/icons/landing_page_html_logo.png")
                .right
                    h4.tit HTML Friendly
                    p.cont Take advantage of unlimited, pay-as-you-go cloud storage with skapi. Whether it's web hosting or serving large files with securi


</template>

<script setup>
import { onMounted } from 'vue';

onMounted(() => {
    document.querySelector('main').classList.add('landing');
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

                img {
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
                    margin: 0 0 40px 0;
                    background: linear-gradient(180deg, #FAFF00 0%, rgba(255, 230, 0, 0.00) 97.55%);
                }
                a {
                    text-decoration: none;
                    font-size: 24px;
                    color: #fff;
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
                    width: 23%;
                    margin-right: 2%;

                    .icon {
                        width: 90px;
                        img {
                            width: 100%;
                            height: 100%;
                        }
                    }
                }
            }
        }
    }
}
</style>