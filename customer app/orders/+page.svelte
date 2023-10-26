<script>
	import {
		BottomNav,
		BottomNavItem,
		Skeleton,
		CardPlaceholder,
		ImagePlaceholder,
		ListPlaceholder,
		Button,
        P,
		Input,
        Radio,
		Helper,
	} from "flowbite-svelte";
	import {
		ArrowLeftToBracketOutline,
		HomeSolid,
		WalletSolid,
		AdjustmentsVerticalOutline,
		UserCircleSolid,
		CloseSolid,
		OpenBookSolid,
		CartPlusSolid,
		CartOutline,
		CartPlusAltSolid,
		ProfileCardOutline,
	} from "flowbite-svelte-icons";
	import { page } from "$app/stores";
	import { browser } from '$app/environment';
	import { enhance } from '$app/forms';
    import { scale, fade } from 'svelte/transition';
    import image from '$lib/images/404.png';
    import image2 from '$lib/images/order.png';
	$: activeUrl = $page.url.pathname;

	let loaded = false;
	let popup = false;
	let notificationVisible = false;
	let notificationTitle;
	let notificationText;
	let popupType = 0;
	let popupInfo;
	let orders;
	let total = 0;
	let orderComplete = false;

	let phone;
	let firstname;

	if (browser) {
		if (!window.localStorage.getItem('account_token')) {
			window.location.href = "/login";
		}

		try {
			orders = JSON.parse(window.localStorage.getItem('orders'));
			Object.values(orders).forEach(function(order) {
				total += parseInt(order.price) * order.amount;
			});
		}
		catch (err) {
			
		}

		loaded = true;
		popupInfo = {
			1: {
				title: 'What payment method would you like to use?',
				radios: {
					0: {
						title: 'Cash on delivery', desc: "Only option as of now"
					}
				}
			},
			2: {
				title: "Please provide your phone number for contacting",
				input: {
					0: {
						placeholder: '(+27) 000 000 0000',
						type: 'number',
						name: 'phone',
						value: "",
					}
				}
			},
			3:
			{
				title: "Please provide your first name this order should go to",
				input: {
					0: {
						placeholder: 'First Name',
						type: 'text',
						name: 'firstname',
						value: "",
					}
				},
				button: 'Finish'
			}
		}
	}

	function removeItem(event) {
		let card;
		let type;

		if (event.target.id == "card") {
			card = event.target;
			type = card.dataset.type;
		}
		else if (event.target.parentNode.id == "card")
		{
			card = event.target.parentNode;
			type = card.dataset.type;
		}
		else
		{
			card = event.target.parentNode.parentNode;
			type = card.dataset.type;
		}

		if (parseInt(card.querySelector('#amount').innerText) <= 1) {
			window.localStorage.removeItem('orders');
			total = 0;
			card.remove();
		}

		let savedOrders = JSON.parse(window.localStorage.getItem('orders'));
		savedOrders[type]['amount'] = savedOrders[type]['amount'] - 1;
		total = savedOrders[type]['amount'] * savedOrders[type]['price']; 
		card.querySelector('#amount').innerText = savedOrders[type].amount;
		savedOrders = JSON.stringify(savedOrders);
		window.localStorage.setItem('orders', savedOrders);
	}

	function logout() {
		window.localStorage.removeItem('orders');
		window.localStorage.removeItem('account_token');
		window.location.href = "/login";
	}

	function showPopup() {
		popupType += 1;
		popup = true;
	}

	function onInput(event) {
		if (popupType == 2) {
			phone = event.target.value;
		}
		else if (popupType == 3) {
			firstname = event.target.value;
		}
	}

	function changeNotifState(title, text) {
        notificationVisible = !notificationVisible;
        notificationText = text;
        notificationTitle = title;

        setTimeout(function() {
            notificationVisible = !notificationVisible
            window.location.href = "/menu";
        },5000)
    }

	function closeOrderPopup() {
		orderComplete = !orderComplete;
		window.localStorage.removeItem('orders');
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

{#if orderComplete}
	<div class="flex justify-center items-center flex-col fixed bottom-0 top-0 right-0 left-0 bg-black bg-opacity-90 z-[120]">
		<div class="relative bg-white w-[323px] h-[400px] rounded-md">
			<h1 class="text-xl font-bold mx-5 mt-5">Order has been placed</h1>
			<h1 class="text-md font-normal text-gray-600 mx-5 mt-2">Your order has been submitted, further details will be discussed shortly.</h1>
			<div class="flex items-center justify-center mt-5">
				<img alt='order.png' src={image2}/>
			</div>
			<Button on:click={closeOrderPopup} class="absolute left-0 right-0 mx-10 bottom-5 text-md">Ok</Button>
		</div>
	</div>
{/if}

<h1 class="text-2xl font-medium mx-5 mt-5">Orders</h1>
<h2 class="text-md text-gray-500 font-normal mx-5 mt-2">
	Click on an order to remove it from your list
</h2>

{#if !loaded}
	<ListPlaceholder class="mt-10 mb-10 mx-10 space-y-10" />
{:else}
	{#if orders}
		<div class="grid mt-10 grid-rows-1 mx-5">
			{#each Object.entries(orders) as [type, item]}
				{#if item.amount > 0}
					<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
					<div role="caption" on:keypress={null} on:click={removeItem} id="card" data-type={type} class="relative flex mb-5 px-2 py-2 border-gray-200 border-2 shadow-sm rounded-md h-[100px] w-full">
						<img alt="itemThumbnail.png" class="rounded-md w-[100px] h-[auto]" src={item.image}/>
						<div id='amount' class="rounded-2xl w-[30px] h-[30px] bg-blue-600 absolute right-0 top-0 mx-[-15px] my-[-15px] text-center text-lg text-white font-semibold leading-[30px]">{item.amount}</div>
						<div>
							<h1 class="font-semibold text-lg mx-5">{item.name}</h1>
							<h1 class="text-center font-medium rounded-sm text-xs py-0.5 px-2 w-fit mt-1 bg-blue-200 text-blue-600 mx-5">{item.time} mins</h1>
							<h1 class="absolute right-5 top-0 bottom-0 font-medium text-2xl mx-5 leading-[100px]">R{item.price}</h1>
						</div>
					</div>
				{/if}
			{/each}
		</div>
	{:else}
		<div class="flex mt-10 justify-center items-center w-full">
			<img alt="404" src={image} class="w-[246px] h-[185px] opacity-40"/>
		</div>
	{/if}
{/if}

<hr class="fixed bottom-[200px] left-5 right-5 h-[1px] bg-gray-200 border-0 dark:bg-gray-700 z-10"/>
<h1 class="fixed bottom-[150px] left-10 text-xl font-medium mt-10 z-10">Total:</h1>

{#if !loaded}
	<div class="fixed bottom-[150px] right-10 text-xl h-2 w-10 animate-pulse bg-slate-300 rounded font-medium mt-10 mb-3 z-10"></div>
{:else}
	<h1 class="fixed bottom-[150px] right-10 text-xl font-medium mt-10 z-10">R{total}</h1>
{/if}

<div class="fixed left-0 right-0 bottom-[0] h-[200px] bg-white" />

<Button on:click={total > 0 ? showPopup : ''} class="fixed bottom-[80px] left-10 text-md right-10 focus:ring-0">Checkout</Button>

{#if popup}
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div on:click={() => {popup = false; popupType = 0;}} class="fixed inset-0 bg-black opacity-90 z-[100]"></div>
	<div class="fixed bottom-[-20px] px-6 py-3 bg-white left-0 right-0 rounded-2xl h-[300px] z-[110]">
		<h1 class="font-bold text-xl my-5">{popupInfo[popupType].title}</h1>

		{#if popupInfo[popupType].radios}
			{#each Object.values(popupInfo[popupType].radios) as radio}
				<Radio class="font-semibold text-lg">{radio.title}</Radio>
				<Helper id="helper-checkbox-text" class="pl-6">{radio.desc}</Helper>
			{/each}
		{/if}

		{#if popupInfo[popupType].input}
			{#each Object.values(popupInfo[popupType].input) as input}
                <Input id='popupInput' value={input.value} type={input.type} on:input={ onInput } name={input.name} class="text-lg focus:ring-blue-500 focus:border-blue-500" placeholder={input.placeholder} />
			{/each}
		{/if}

		{#if popupType == 3}
			<form method="POST" action="?/order" use:enhance={({ formElement, formData, action, cancel, submitter }) => {
				return async ({ result }) => {
					if (result.data.status == "success") {
						popup = false;
						orderComplete = true;
					}
					else if (result.data.status == "invalid") {
						// invalid account token
						changeNotifState('Fail', "Your account is not registered. Please logout");
					}
					else if (result.data.status == "creds")
					{
						// didnt provide all info
						changeNotifState('Fail', 'We need all credentials including first name and phone number');
					}
					else 
					{
						// fail
						changeNotifState('Fail', 'Something went wrong. Please try again later');
					}
				};
			}}>
				<Input class="hidden" name='orders' value={window.localStorage.getItem('orders')}/>
				<Input class="hidden" name='phone' value={phone}/>
                <Input class="hidden" name='firstname' value={firstname}/>
				<Input class="hidden" name='account_token' value={window.localStorage.getItem('account_token')}/>
				<Button type="submit" class="absolute bottom-10 text-md">{popupInfo[popupType].button !== undefined? popupInfo[popupType].button: 'Continue'}</Button>
			</form>
		{:else}
			<Button on:click={showPopup} class="absolute bottom-10 text-md">{popupInfo[popupType].button !== undefined? popupInfo[popupType].button: 'Continue'}</Button>
		{/if}

		<h1 class="absolute right-10 bottom-12 font-bold text-md">{popupType}/3</h1>
	</div>
{/if}

<BottomNav
	{activeUrl}
	position="fixed"
	classInner="grid-cols-3"
	classActive="text-primary-500 hover:text-primary-500 dark:hover:text-primary-500 dark:text-primary-500"
>
	<BottomNavItem btnName="Menu" href="/menu">
		<OpenBookSolid
			class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500"
		/>
	</BottomNavItem>
	<BottomNavItem btnName="Orders" href="/orders">
		<CartPlusAltSolid
			class="focus:border-0 w-5 h-5 mb-1 dark:text-gray-400 active:text-primary-500 group-hover:text-primary-500 dark:group-hover:text-primary-500"
		/>
	</BottomNavItem>
	<BottomNavItem btnName="logout" on:click={logout}>
		<ArrowLeftToBracketOutline class="focus:border-0 w-5 h-5 mb-1 text-gray-500 dark:text-gray-400 group-hover:text-primary-500 dark:group-hover:text-primary-500"/>
	</BottomNavItem>
</BottomNav>
