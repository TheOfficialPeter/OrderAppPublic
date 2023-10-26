<script>
    import image from '$lib/images/banner2.png'
    import { Label, Input, Button, GradientButton, Checkbox  } from 'flowbite-svelte';
	import { browser } from '$app/environment';
    import { enhance } from '$app/forms';
    import { scale } from 'svelte/transition';
    import { CloseSolid } from 'flowbite-svelte-icons';

    $: type = show_password ? 'text': 'password';
    let show_password = false;
    let passwordValue = '';
    let repeatValue = '';

    let notificationTitle = "";
    let notificationText = "";
    let notificationVisible = false;

    function onRepeat(event) {
        passwordValue = event.target.value;
    }

    function onPassword(event) {
        repeatValue = event.target.value;
    }

    function changeNotifState(title, text) {
        notificationVisible = !notificationVisible;
        notificationText = text;
        notificationTitle = title;

        setTimeout(function() {
            notificationVisible = !notificationVisible
            window.location.href = "/login";
        },5000)
    }

    let urlEmail = "";
    let urlResetToken = "";

    if (browser) {
        const urlParams = new URLSearchParams(window.location.search);
        const email = urlParams.get('email');
        const reset_token = urlParams.get('reset_token');

        urlEmail = email;
        urlResetToken = reset_token;
    }
</script>

{#if notificationVisible}
<div transition:scale class="fixed z-[200] bg-indigo-950 top-5 left-5 right-5 rounded-md">
    <h1 class="mx-5 mt-4 text-blue-500 text-lg font-medium">{notificationTitle}</h1>
    <h1 class="mx-5 mt-2 mb-5 text-blue-800 text-sm font-medium">{notificationText}</h1>
    <CloseSolid on:click={changeNotifState} size="sm" color="#3f83f880" class="absolute top-5 right-5"></CloseSolid>
</div>
{/if}

<div class="flex justify-center inset-0">
    <img src={image} alt="banner.png" class="absolute top-0 w-[283px] h-[283px]"/>
</div>

<div class="flex justify-center absolute top-[300px] left-0 right-0 bottom-0 min-h-[400px]">
    <form method="POST" action="?/forgot" class="w-[290px]" use:enhance={({ formElement, formData, action, cancel }) => {
            return async ({ result }) => {
                if (result.data.status == "success") {
                    changeNotifState('Success', "Your password has been reset.");
                }
                else if (result.data.status == "link") {
                    changeNotifState('Fail', "The link you are using is invalid.");
                }
                else {
                    changeNotifState('Fail', "Passwords do not match.");
                }
            };
        }}>

        <Input value={urlEmail} name='urlEmail' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500 hidden"/>
        <Input value={urlResetToken} name='urlResetToken' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500 hidden"/>

        <div class="mb-6">
            <Label for="large-input" class="block text-md mb-2">New Password</Label>
            <Input required on:input={ onPassword } {type} {passwordValue} name='password' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="password"/>
        </div>

        <div class="mb-6">
            <Label for="large-input" class="block text-md mb-2">Repeat Password</Label>
            <Input required on:input={ onRepeat } {type} {repeatValue} name='repeat' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="password"/>
        </div>

        <div>
            <Checkbox on:click="{ () => show_password = !show_password }">{show_password ? 'Hide' : 'Show'} password</Checkbox>
        </div>

        <Button type='submit' color="blue" class="absolute bottom-[50px] font-medium text-lg focus:ring-2 w-[290px]">Change password</Button>
    </form>
</div>