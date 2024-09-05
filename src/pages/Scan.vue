<!-- refresh vue skill -->
<script setup>
    import {
        ref
    } from 'vue';
    import axios from 'axios';

    const chapDomainsThatUseForScamsInIndonesia = [
        'xyz',
        'top',
        'club',
        'info',
        'oline',
        'site',
        'tech',
        'live',
        'pace',
        'work',
    ];

    const blueAlerts = ref([]);
    const warningAlerts = ref([]);
    const dangerAlerts = ref([]);

    const isUrlNotFound = ref(false);
    const isUrlFound = ref(false);

    const linkValue = ref('');

    const domainPoints = ref(100);
    const isSubmit = ref(false);
    const isProcessing = ref(false);

    function isUrl(url) {
        try {
            new URL(url);
            return true;
        } catch (e) {
            return false;
        }
    }

    async function virtualVisitDomain(url){
        let response = await axios.get(url);
        return response;
    }

    async function scanLink()
    {
        isProcessing.value = true;
        //* Checking Rule
        // 1. Check is url found
        // 2. Check is valid url
        // 3. Check is http or https
        // 4. Check is domain redirected
        // 5. Check is domain that often used for scam
        // 6. Check is redirected to another domain

        if (linkValue.value === '') {
            isUrlNotFound.value = true;
            isUrlFound.value = false;
        } else {
            isUrlNotFound.value = false;
            isUrlFound.value = true;
        }

        //* check is valid url
        if (!isUrl(linkValue.value)) {
            isUrlNotFound.value = true;
            isUrlFound.value = false;
            return;
        }
        
        //* virtual scan the damn link
        let response = await axios.get(linkValue.value);
        
        if (response.status === 404) {
            isUrlNotFound.value = true;
            isUrlFound.value = false;
            return;
        }

        //* domain redirected
        let domains = [
            linkValue.value.split('/')[2],
        ];

        //* check is scam link
        const isUseDomainScam = chapDomainsThatUseForScamsInIndonesia.some((domain) => linkValue.value.includes(domain));
        if(isUseDomainScam){
            domainPoints.value -= 50;
            dangerAlerts.value.push({
                title: 'This link using domain that often used for scam',
                description: 'This domain is very cheap and often used for scamming people. If a company uses this domain, it is very likely that the company is a scam because they don\'t want to spend money on a good domain.',
            });
        }else{
            blueAlerts.value.push({
                title: 'This link use a good domain',
                description: 'This domain is a good domain. This is a good sign that the company is legitimate.',
            });
        }

        //* check is redirected after get request
        if(response.request.responseURL !== linkValue.value){
            //* check is still http or https
            if(response.request.responseURL.startsWith('http://')){
                domainPoints.value -= 45;
                dangerAlerts.value.push({
                    title: 'This link is not secure',
                    description: 'This link is not secure because it uses http instead of https. This means that the data you send to this website can be intercepted by other people.',
                });
            }else{
                blueAlerts.value.push({
                    title: 'This link is secure',
                    description: 'This link is secure because it uses https. This means that the data you send to this website is encrypted and cannot be intercepted by other people.',
                });
            }

            //* check is redirected to another domain
            if(!response.request.responseURL.includes(domains[0])){
                domainPoints.value -= 5;
                let domainThatRedirected = response.request.responseURL.split('/')[2];
                let urlThatRedirected = response.request.responseURL;

                domains.push(domainThatRedirected);

                //* visit again the redirected domain, loop while
                let isRedirected = true;
                while(isRedirected){
                    response = await virtualVisitDomain(urlThatRedirected);
                    if(response.request.responseURL.includes(domains[0])){
                        isRedirected = false;
                    }else{
                        urlThatRedirected = response.request.responseURL;
                        domainThatRedirected = response.request.responseURL.split('/')[2];
                        
                        domains.push(domainThatRedirected);
                    }
                }

                warningAlerts.value.push({
                    title: 'This link is redirected to another domain',
                    description: 'This link is redirected to another domain. This is a common tactic used by scammers to trick people into thinking they are on a legitimate website.',
                });
            }else{
                blueAlerts.value.push({
                    title: 'This link is not redirected to another domain',
                    description: 'This link is not redirected to another domain. This is a good sign that the company is legitimate.',
                });
            }
        }else{
            blueAlerts.value.push({
                title: 'This link is not redirected',
                description: 'This link is not redirected. This is a good sign that the company is legitimate.',
            });
        }

        isSubmit.value = true;
        isProcessing.value = false;
    }
    function resetLink()
    {
        linkValue.value = '';
        isUrlNotFound.value = false;
        isUrlFound.value = false;
        domainPoints.value = 100;
        blueAlerts.value = [];
        warningAlerts.value = [];
        dangerAlerts.value = [];

        isSubmit.value = false;
        isProcessing.value = false;
    }
</script>
<template>
    <main class="grid min-h-full place-items-center bg-white px-6 py-24 sm:py-32 lg:px-8">
        <div class="text-center">
            <p class="text-base font-semibold text-indigo-600">
                Kita membantu kamu untuk mengecek keamanan link yang ingin kamu kunjungi 🐧
            </p>
            <h1 class="mt-4 text-3xl font-bold tracking-tight text-gray-900 sm:text-5xl">
                Scan Disini
            </h1>
            <div class="mt-6">
                <div class="mt-2">
                    <input 
                    v-model="linkValue"
                    type="text" name="link" id="link"
                        class="block w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"
                        placeholder="Masukkan link yang ingin kamu scan">
                </div>
                <button 
                v-if="!isProcessing"
                @click="scanLink"
                type="button" class="mt-4 rounded-md bg-indigo-50 px-3.5 py-2.5 text-sm font-semibold text-indigo-600 shadow-sm hover:bg-indigo-100">
                    Scan URL 🕵️
                </button>

                <!-- loader when processing -->
                <div v-else class="mt-4">
                    <svg class="animate-spin h-5 w-5 text-indigo-600 mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V4a10 10 0 00-10 10h2zm2 8a8 8 0 018-8h2a10 10 0 00-10-10v2zm8 2a8 8 0 01-8-8h-2a10 10 0 0010 10v-2zm-8-2a8 8 0 018 8v2a10 10 0 00-10-10h2z"></path>
                    </svg>
                </div>
            </div>

            <p v-if="isUrlNotFound" class="mt-6 text-base leading-7 text-gray-600">
                Maaf, link yang kamu masukkan tidak valid 😢
            </p>
            <div class="mt-10 flex items-center justify-center gap-x-6">
                <a 
                @click="resetLink"
                v-if="isUrlFound" href="/#"
                    class="rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">
                    Reset
                </a>
            </div>

            <!-- show link points -->
            <div v-if="linkValue !== '' && isSubmit" class="mt-10">
                <h2 class="text-xl font-semibold text-gray-900">
                    Point Link
                </h2>
                <div class="mt-4 spacex-y-4">
                    <!-- jika lebih atau sama dengan 95, hijau -->
                    <div v-if="domainPoints >= 95"
                    class="bg-green-50 p-4 rounded-md">
                        <h3 class="text-lg font-semibold text-green-800">
                            {{ domainPoints }} Points
                        </h3>
                        <p class="mt-2 text-base text-green-600">
                            Link ini aman untuk dikunjungi.
                        </p>
                    </div>

                    <!-- jika kurang dari 95 dan lebih atau sama dengan 50, kuning -->
                    <div v-else-if="domainPoints < 95 && domainPoints >= 50"
                    class="bg-yellow-50 p-4 rounded-md">
                        <h3 class="text-lg font-semibold text-yellow-800">
                            {{ domainPoints }} Points
                        </h3>
                        <p class="mt-2 text-base text-yellow-600">
                            Link ini tidak sepenuhnya aman untuk dikunjungi, kalau mau tetap kunjungi, pakai tab incognito/tab aman ya. Lebih bagus lagi kalau pakai browser yang aman seperti DuckDuckGo atau Brave.
                        </p>
                    </div>

                    <!-- jika kurang dari 50, merah -->
                    <div v-else-if="domainPoints < 50"
                    class="bg-red-50 p-4 rounded-md">
                        <h3 class="text-lg font-semibold text-red-800">
                            {{ domainPoints }} Points
                        </h3>
                        <p class="mt-2 text-base text-red-600">
                            Link ini tidak aman untuk dikunjungi, sebaiknya jangan kunjungi link ini. Jika kamu sudah mengunjungi link ini, segera tutup tab tersebut dan bersihkan cache browser kamu.
                        </p>
                    </div>
                </div>
            </div>

            <!-- foreach blue alerts -->
            <div v-if="blueAlerts.length > 0" class="mt-10">
                <h2 class="text-xl font-semibold text-gray-900">
                    Info Point
                </h2>
                <div class="mt-4 space-y-4">
                    <div v-for="alert in blueAlerts"
                    :key="alert.title"
                    class="bg-blue-50 p-4 rounded-md">
                        <h3 class="text-lg font-semibold text-blue-800">
                            {{ alert.title }}
                        </h3>
                        <p class="mt-2 text-base text-blue-600">
                            {{ alert.description }}
                        </p>
                    </div>
                </div>
            </div>

            <!-- foreach warning alerts -->
            <div v-if="warningAlerts.length > 0" class="mt-10">
                <h2 class="text-xl font-semibold text-gray-900">
                    Perlu Perhatian
                </h2>
                <div class="mt-4 space-y-4">
                    <div v-for="alert in warningAlerts"
                    :key="alert.title"
                    class="bg-yellow-50 p-4 rounded-md">
                        <h3 class="text-lg font-semibold text-yellow-800">
                            {{ alert.title }}
                        </h3>
                        <p class="mt-2 text-base text-yellow-600">
                            {{ alert.description }}
                        </p>
                    </div>
                </div>
            </div>

            <!-- foreach danger alerts -->
            <div v-if="dangerAlerts.length > 0" class="mt-10">
                <h2 class="text-xl font-semibold text-gray-900">
                    Bahaya! 🚨
                </h2>
                <div class="mt-4 space-y-4">
                    <div v-for="alert in dangerAlerts"
                    :key="alert.title"
                    class="bg-red-50 p-4 rounded-md">
                        <h3 class="text-lg font-semibold text-red-800">
                            {{ alert.title }}
                        </h3>
                        <p class="mt-2 text-base text-red-600">
                            {{ alert.description }}
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </main>

</template>
<style scoped>

</style>