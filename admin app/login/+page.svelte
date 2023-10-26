<script>
    import image from '$lib/images/banner1.png';
    import { Label, Input, Button, GradientButton, Checkbox  } from 'flowbite-svelte';
	import { browser } from '$app/environment';
    import { enhance } from '$app/forms';
    import { scale } from 'svelte/transition';
    import { CloseSolid } from 'flowbite-svelte-icons';

    $: type = show_password ? 'text' : 'password';
    let value = "";
    let show_password = false;
    
    let notificationTitle = "";
    let notificationText = "";
    let notificationVisible = false;

    let email_value = "";

    function onInput (event) {
        value = event.target.value
    }

    function changeNotifState(title, text) {
        notificationVisible = !notificationVisible;
        notificationText = text;
        notificationTitle = title;

        setTimeout(function() {
            notificationVisible = !notificationVisible
        },5000)
    }

    if (browser) {
        let account_token = window.localStorage.getItem('account_token');
        if (account_token != undefined) {
            window.location.href = "/menu";
        }
    }

    function change_email(event) {
        email_value = event.target.value;
    }

</script>

<img src={image} alt="banner.png" class="absolute inset-x-0 top-0 w-full h-[250px]"/>

{#if notificationVisible}
<div transition:scale class="fixed z-[200] bg-indigo-950 top-5 left-5 right-5 rounded-md">
    <h1 class="mx-5 mt-4 text-blue-500 text-lg font-medium">{notificationTitle}</h1>
    <h1 class="mx-5 mt-2 mb-5 text-blue-800 text-sm font-medium">{notificationText}</h1>
    <CloseSolid on:click={changeNotifState} size="sm" color="#3f83f880" class="absolute top-5 right-5"></CloseSolid>
</div>
{/if}

<div class="absolute right-0 left-0 bottom-0 top-[35vh] flex justify-center">
    <div class="relative w-[290px] min-h-[400px]">
        <form method="POST" action="?/login" use:enhance={({ formElement, formData, action, cancel }) => {
            return async ({ result }) => {
                if (result.data.status == "success") {
                    window.localStorage.setItem('account_token', result.data.account_token);
                    window.location.href = "/menu";
                }
                else if (result.data.status == "fail") {
                    changeNotifState('Fail', 'The email or password provided is incorrect');
                }
                else if (result.data.status == "verify") {
                    changeNotifState('Success', "A verification link has been sent to your email");
                }
                else if (result.data.status == "auth") {
                    changeNotifState('Fail', "This account is not authorized to use this app");
                }
            };
        }}>
            <div class="mb-6">
                <Label for="large-input" class="block text-md mb-2">Email</Label>
                <Input required type="email" name='email' id="email" {email_value} on:input={change_email} size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="example@example.com"/>
            </div>

            <div class="mb-6">
                <Label for="large-input" class="block text-md mb-2">Password</Label>
                <Input required on:input={ onInput } {type} {value} name='password' id="password" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="password" />
            </div>

            <div>
                <Checkbox on:click="{ () => show_password = !show_password }">{show_password ? 'Hide' : 'Show'} password</Checkbox>
            </div>

            <Button type='submit' color="blue" class="absolute bottom-[50px] right-0 left-0 font-medium text-lg focus:ring-2">Sign in</Button>
        </form>

        <form method="POST" action="?/forgot" use:enhance={({ formElement, formData, action, cancel }) => {
            return async ({ result }) => {
                if (result.data.status == "success") {
                    changeNotifState('Success', "A link has been sent to your email to reset your password");
                }
                else if (result.data.status == "creds") {
                    changeNotifState('Fail', "This account is not authorized to use this app");
                }
                else {
                    changeNotifState('Fail', "Something went wrong. Please try again later.");
                }
            };
        }}>
            <Input required type="email" name='email' value={email_value} size="lg" class="hidden text-lg focus:ring-blue-500 focus:border-blue-500"/>
            <button type="submit" class="absolute bg-transparent rounded-none bottom-[15px] py-0 focus:bg-transparent active:bg-transparent left-0 right-0 text-center text-black-500 underline font-regular">Forgot password?</button>
        </form>
    </div>
</div>