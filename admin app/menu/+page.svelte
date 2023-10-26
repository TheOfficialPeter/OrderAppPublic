<script>
	import {
		ImagePlaceholder,
		Indicator,
		CardPlaceholder,
		TextPlaceholder,
		BottomNav,
		Button,
		Input,
		BottomNavItem,
		Skeleton,
		Img,
		Card,
	} from "flowbite-svelte";
	import {
		HomeSolid,
		WalletSolid,
		AdjustmentsVerticalOutline,
		UserCircleSolid,
		OpenBookSolid,
		CloseSolid,
		CartPlusSolid,
		PlusSolid,
		CartOutline,
		CartPlusAltSolid,
		ProfileCardOutline,
		ArrowLeftToBracketOutline,
        EnvelopeOpenOutline
	} from "flowbite-svelte-icons";
	import { page } from "$app/stores";
	import { browser } from '$app/environment';
    import { scale } from 'svelte/transition';
    import image from '$lib/images/404.png';
    import { enhance } from '$app/forms';

	let activeUrl = $page.url.pathname;
	let loaded = false;
	let items;
	let notificationTitle = "";
    let notificationText = "";
    let notificationVisible = false;
	let messageBoxVisible = false;
	let itemToRemove = 0;
	let account_token;
	
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

			fetch(`/menu?account_token=${account_token}`, {
				method: "GET"
			}).then(async function(response) {
				response = await response.json();
				
				if (response.status == "fail") {
					changeNotifState('Fail', "Your account is not registered. Please logout");
				}

				items = response.result;
				loaded = true;
			})
		}
	}

	function logout() {
		window.localStorage.removeItem('orders');
		window.localStorage.removeItem('account_token');
		window.location.href = "/login";
	}

	function openAdd() {
		window.location.href = "/add";
	}

	function changeMessageBoxState(event) {
		try {
			let card = event.target.parentNode;

			if (card && card.id == "card") {
				itemToRemove = card.dataset.id;
			}
		}
		catch {

		}

		messageBoxVisible = !messageBoxVisible;
	}
</script>

{#if notificationVisible}
	<div transition:scale class="fixed z-[200] bg-indigo-950 top-5 left-5 right-5 rounded-md">
		<h1 class="mx-5 mt-4 text-blue-500 text-lg font-medium">{notificationTitle}</h1>
		<h1 class="mx-5 mt-2 mb-5 text-blue-800 text-sm font-medium">{notificationText}</h1>
		<CloseSolid on:click={changeNotifState} size="sm" color="#3f83f880" class="absolute top-5 right-5"></CloseSolid>
	</div>
{/if}

{#if messageBoxVisible}
	<form action="?/remove" method="POST" use:enhance={({ formElement, formData, action, cancel }) => {
            return async ({ result }) => {
                if (result.data.status == "success") {
					changeMessageBoxState();
					changeNotifState('Success', 'Removed item from menu');
                }
                else if (result.data.status == "fail") {
					changeMessageBoxState();
                    changeNotifState('Fail', 'Could not remove item. Please try again later');
                }
                else if (result.data.status == "creds") {
					changeMessageBoxState();
                    changeNotifState('Fail', 'You account is not authorized to remove items OR retry clicking the remove button');
                }
				else if (result.data.status == "invalid") {
					changeMessageBoxState();
					changeNotifState('Fail', 'Your account is not authorized to remove items from the menu. Try signing in again')
				}
            };
        }}>
		<div class="flex justify-center items-center flex-col fixed bottom-0 top-0 right-0 left-0 bg-black bg-opacity-90 z-[120]">
			<div class="relative bg-white w-[323px] h-[200px] rounded-md">
				<Input class="hidden" name='itemId' value={itemToRemove}/>
				<Input class="hidden" name='account_token' value={account_token}/>
				<h1 class="text-xl font-bold mx-5 mt-5">Are you sure?</h1>
				<h1 class="text-md font-normal text-gray-600 mx-5 mt-2">Are you sure you want to remove this item from the menu?</h1>
				<div class="absolute bottom-5 left-0 right-0 flex items-center justify-center mx-5">
					<Button outline=true on:click={changeMessageBoxState} class="mx-2 bottom-5 text-md">Cancel</Button>
					<Button type="submit" name='remove' class="mx-2 bottom-5 text-md">Remove</Button>
				</div>
			</div>
		</div>
	</form>
{/if}

<h1 class="text-2xl font-medium mx-5 mt-10">Menu</h1>
<h1 class="text-md text-gray-500 font-normal mx-5 mt-2">Add or remove items from the menu</h1>

{#if loaded == true && items == undefined}
	<div class="flex mt-10 justify-center items-center w-full">
		<img alt="404" src={image} class="w-[246px] h-[185px] opacity-40"/>
	</div>
{/if}

<div class="grid grid-cols-2 gap-5 mx-5 justify-center mt-10 mb-[30vh]">
	{#if !loaded}
		<div class="px-3 py-3 bg-slate-200 rounded-md w-full h-[200px] animate-pulse"></div>
		<div class="px-3 py-3 bg-slate-200 rounded-md w-full h-[200px] animate-pulse"></div>
		<div class="px-3 py-3 bg-slate-200 rounded-md w-full h-[200px] animate-pulse"></div>
		<div class="px-3 py-3 bg-slate-200 rounded-md w-full h-[200px] animate-pulse"></div>
		<div class="px-3 py-3 bg-slate-200 rounded-md w-full h-[200px] animate-pulse"></div>
		<div class="px-3 py-3 bg-slate-200 rounded-md w-full h-[200px] animate-pulse"></div>
	{:else}
		{#if items !== undefined}
			{#each items as item}
				<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
				<div role="contentinfo" on:keypress={null} id='card' data-id={item.itemid} data-name={item.itemname} data-price={item.itemprice} data-imageurl={item.itemimageurl} data-time={item.itemtime} class="rounded-md px-2 py-2 w-full h-min border-gray-200 border-2 relative">
					<img alt="MenuItem.png" class="rounded-md" src={item.itemimageurl}/>
					<h1 class="font-medium text-lg mt-3 mb-2">{item.itemname}</h1>
					<h1 class="font-medium text-xs mb-2 text-blue-500 px-2 rounded-md py-1 bg-blue-100 w-fit">{item.itemtime} mins</h1>
					<h1 class="font-semibold text-xl mb-2 w-fit">R{item.itemprice}</h1>
					<Button on:click={changeMessageBoxState} class="w-full px-2 text-md">Remove</Button>
				</div>
			{/each}
		{/if}
	{/if}
</div>

<Button on:click={openAdd} shadow=true pill=true class="h-[50px] w-[50px] fixed right-10 bottom-24 focus:ring-0 focus:outline-0 focus:border-0 hover:bg-none"><PlusSolid class="focus:ring-0 focus:bg-transparent focus:outline-0 focus:border-0"></PlusSolid></Button>

<BottomNav {activeUrl} position="fixed" classInner="grid-cols-3" classActive="text-primary-500 hover:text-primary-500 dark:hover:text-primary-500 dark:text-primary-500" >
	<BottomNavItem btnName="Menu" href="/menu">
		<OpenBookSolid class="focus:border-0 w-5 h-5 mb-1 dark:text-gray-400 active:text-primary-500 group-hover:text-primary-500 dark:group-hover:text-primary-500" />
	</BottomNavItem>
	<BottomNavItem btnName="Orders" href="/orders">
		<CartPlusAltSolid class="focus:border-0 w-5 h-5 mb-1 focus:outline-0 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500" />
	</BottomNavItem>
	<BottomNavItem btnName="Logout" on:click={logout}>
		<ArrowLeftToBracketOutline class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500"/>
	</BottomNavItem>
</BottomNav>
