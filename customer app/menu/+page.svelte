<script>
	import {
		ImagePlaceholder,
		Indicator,
		CardPlaceholder,
		TextPlaceholder,
		BottomNav,
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
		CartOutline,
		CartPlusAltSolid,
		ProfileCardOutline,
		ArrowLeftToBracketOutline
	} from "flowbite-svelte-icons";
	import { page } from "$app/stores";
	import { browser } from '$app/environment';
    import { scale } from 'svelte/transition';
    import image from '$lib/images/404.png';

	$: activeUrl = $page.url.pathname;

	let loaded = false;
	let items;
	let notificationTitle = "";
    let notificationText = "";
    let notificationVisible = false;

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
			// Load menu items
			fetch(`/menu?account_token=${window.localStorage.getItem('account_token')}`, {
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

	function checkForOrdersWithID(id) {
		try {
			let orders = JSON.parse(window.localStorage.getItem('orders'))[id];
			return orders;
		}
		catch {
			return false;
		}
	}

	function addToOrder(event) {
		let card;
		if (event.target.id == "card") {
			card = event.target;

			if (checkForOrdersWithID(card.dataset.id) == false) {
				if (window.localStorage.getItem('orders') !== null) {
					let order = JSON.parse(window.localStorage.getItem('orders'));
					order[card.dataset.id] = {'id': card.dataset.id, 'name': card.dataset.name, 'price': card.dataset.price, 'image': card.dataset.imageurl, 'time': card.dataset.time, 'amount': 1};
					window.localStorage.setItem('orders', JSON.stringify(order));
				}
				else
				{
					let order = {};
					order[card.dataset.id] = {'id': card.dataset.id, 'name': card.dataset.name, 'price': card.dataset.price, 'image': card.dataset.imageurl, 'time': card.dataset.time, 'amount': 1};
					window.localStorage.setItem('orders', JSON.stringify(order));
				}
			}
			else
			{
				let orders = JSON.parse(window.localStorage.getItem('orders'));

				orders[card.dataset.id]['amount'] = orders[card.dataset.id]['amount'] + 1;
				window.localStorage.setItem('orders', JSON.stringify(orders));
			}
		}
		else
		{
			card = event.target.parentNode;

			if (checkForOrdersWithID(card.dataset.id) == false) {
				if (window.localStorage.getItem('orders') !== null) {
					let order = JSON.parse(window.localStorage.getItem('orders'));
					order[card.dataset.id] = {'id': card.dataset.id, 'name': card.dataset.name, 'price': card.dataset.price, 'image': card.dataset.imageurl, 'time': card.dataset.time, 'amount': 1};
					window.localStorage.setItem('orders', JSON.stringify(order));
				}
				else
				{
					let order = {};
					order[card.dataset.id] = {'id': card.dataset.id, 'name': card.dataset.name, 'price': card.dataset.price, 'image': card.dataset.imageurl, 'time': card.dataset.time, 'amount': 1};
					window.localStorage.setItem('orders', JSON.stringify(order));
				}
			}
			else
			{
				let orders = JSON.parse(window.localStorage.getItem('orders'));

				orders[card.dataset.id]['amount'] = orders[card.dataset.id]['amount'] + 1;
				window.localStorage.setItem('orders', JSON.stringify(orders));
			}
		}
	}
	
	function logout() {
		window.localStorage.removeItem('orders');
		window.localStorage.removeItem('account_token');
		window.location.href = "/login";
	}
</script>


{#if notificationVisible}
	<div transition:scale class="fixed z-[200] bg-indigo-950 top-5 left-5 right-5 rounded-md">
		<h1 class="mx-5 mt-4 text-blue-500 text-lg font-medium">{notificationTitle}</h1>
		<h1 class="mx-5 mt-2 mb-5 text-blue-800 text-sm font-medium">{notificationText}</h1>
		<CloseSolid on:click={changeNotifState} size="sm" color="#3f83f880" class="absolute top-5 right-5"></CloseSolid>
	</div>
{/if}

<h1 class="text-2xl font-medium mx-5 mt-10">Menu</h1>
<h1 class="text-md text-gray-500 font-normal mx-5 mt-2">Pick an item to add to your order</h1>

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
				<div role="contentinfo" on:keypress={null} id='card' data-id={item.itemid} data-name={item.itemname} data-price={item.itemprice} data-imageurl={item.itemimageurl} data-time={item.itemtime} on:click={addToOrder} class="rounded-md hover:border-blue-500 transition-all ease-in-out px-2 py-2 w-full h-min border-gray-200 border-2">
					<img alt="Burger.png" class="rounded-md" src={item.itemimageurl}/>
					<h1 class="font-medium text-lg mt-3 mb-2">{item.itemname}</h1>
					<h1 class="font-medium text-xs mb-2 text-blue-500 px-2 rounded-md py-1 bg-blue-100 w-fit">{item.itemtime} mins</h1>
					<h1 class="font-semibold text-xl mb-2 w-fit">R{item.itemprice}</h1>
				</div>
			{/each}
		{/if}
	{/if}
</div>

<BottomNav {activeUrl} position="fixed" classInner="grid-cols-3" classActive="text-primary-500 hover:text-primary-500 dark:hover:text-primary-500 dark:text-primary-500" >
	<BottomNavItem btnName="Menu" href="/menu">
		<OpenBookSolid class="focus:border-0 w-5 h-5 mb-1 dark:text-gray-400 active:text-primary-500 group-hover:text-primary-500 dark:group-hover:text-primary-500" />
	</BottomNavItem>
	<BottomNavItem btnName="Orders" href="/orders">
		<CartPlusAltSolid class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500" />
	</BottomNavItem>
	<BottomNavItem btnName="Logout" on:click={logout}>
		<ArrowLeftToBracketOutline class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500"/>
	</BottomNavItem>
</BottomNav>
