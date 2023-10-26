<script>
	import { browser } from '$app/environment';
    import { scale } from 'svelte/transition';
	import { CloseSolid, ArrowLeftSolid, OpenBookSolid, CartPlusAltSolid, ArrowLeftToBracketOutline, ArrowLeftOutline } from "flowbite-svelte-icons";
	import { BottomNav, Input, Label, Button, BottomNavItem } from "flowbite-svelte";
	import { page } from "$app/stores";
    import { enhance } from '$app/forms';

	$: activeUrl = $page.url.pathname;

    let notificationVisible = false;
    let notificationTitle;
	let account_token;
    let notificationText;

    function changeNotifState(title, text) {
        notificationVisible = !notificationVisible;
        notificationText = text;
        notificationTitle = title;

        setTimeout(function() {
            notificationVisible = !notificationVisible
        },5000)
    }

    if (browser) {
		if (window.localStorage.getItem('account_token') == null) {
			window.location.href = "/login";
		}
		else
		{
			account_token = window.localStorage.getItem('account_token');
		}
	}

	function logout() {
		window.localStorage.removeItem('orders');
		window.localStorage.removeItem('account_token');
		window.location.href = "/login";
	}

	function goBack() {
		window.location.href = "/menu";
	}

</script>

{#if notificationVisible}
	<div transition:scale class="fixed z-[200] bg-indigo-950 top-5 left-5 right-5 rounded-md">
		<h1 class="mx-5 mt-4 text-blue-500 text-lg font-medium">{notificationTitle}</h1>
		<h1 class="mx-5 mt-2 mb-5 text-blue-800 text-sm font-medium">{notificationText}</h1>
		<CloseSolid on:click={changeNotifState} size="sm" color="#3f83f880" class="absolute top-5 right-5"></CloseSolid>
	</div>
{/if}

<div class="mx-8 mt-10">
	<ArrowLeftOutline on:click={goBack} color="#1A56DB" class="inline mb-2 focus:outline-0"></ArrowLeftOutline>
	<h1 class="text-2xl font-medium inline mx-5">Add item</h1>
	<h1 class="text-md text-gray-500 font-normal mx-10 mt-2">Provide information for this item</h1>
</div>

<div class="absolute bottom-0 right-0 left-0 top-28">
	<form action="?/add" method="POST" use:enhance={({ formElement, formData, action, cancel }) => {
            return async ({ result }) => {
                if (result.data.status == "success") {
					changeNotifState('Success', 'Added item to menu');
                }
                else if (result.data.status == "fail") {
                    changeNotifState('Fail', 'Could not add item. Please try again later');
                }
                else if (result.data.status == "creds") {
                    changeNotifState('Fail', 'You account is not authorized to add items OR retry clicking the Add item button');
                }
				else if (result.data.status == "invalid") {
					changeNotifState('Fail', 'Your account is not authorized to add items to the menu. Try signing in again')
				}
            };
        }}>
		<div class="mb-6 mx-10 mt-10">
			<Label for="large-input" class="block text-md mb-2">Item name</Label>
			<Input required type="text" name='itemName' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="Item name here"/>
		</div>

		<div class="mb-6 mx-10 mt-5">
			<Label for="large-input" class="block text-md mb-2">Item price</Label>
			<Input required type="number" name='itemPrice' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="0.00"/>
		</div>

		<div class="mb-6 mx-10 mt-5">
			<Label for="large-input" class="block text-md mb-2">Item image url</Label>
			<Input required type="text" name='itemImage' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="Item image url here"/>
		</div>

		<div class="mb-6 mx-10 mt-5">
			<Label for="large-input" class="block text-md mb-2">Item preparation time</Label>
			<Input required type="number" name='itemTime' size="lg" class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder="0 minutes"/>
		</div>

		<div class="w-full h-[40vh]"></div>

		<Input name='account_token' value={account_token}/>
		<Button type='submit' color="blue" class="fixed bottom-[100px] right-10 left-10 font-medium text-lg focus:ring-2">Add item</Button>
	</form>
</div>

<BottomNav {activeUrl} position="fixed" classInner="grid-cols-3" classActive="text-primary-500 hover:text-primary-500 dark:hover:text-primary-500 dark:text-primary-500" >
	<BottomNavItem btnName="Menu" href={activeUrl}>
		<OpenBookSolid class="focus:border-0 w-5 h-5 mb-1 dark:text-gray-400 active:text-primary-500 group-hover:text-primary-500 dark:group-hover:text-primary-500" />
	</BottomNavItem>
	<BottomNavItem btnName="Orders" href="/orders">
		<CartPlusAltSolid class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500" />
	</BottomNavItem>
	<BottomNavItem btnName="Logout" on:click={logout}>
		<ArrowLeftToBracketOutline class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500"/>
	</BottomNavItem>
</BottomNav>