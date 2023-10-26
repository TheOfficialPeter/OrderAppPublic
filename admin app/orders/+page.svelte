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
		CaretDownSolid,
		WalletSolid,
		AdjustmentsVerticalOutline,
		UserCircleSolid,
		CloseSolid,
		OpenBookSolid,
		CartPlusSolid,
		CartOutline,
		CartPlusAltSolid,
		ProfileCardOutline,
        AngleDownSolid,
        OrdoredListOutline,
        CloseCircleSolid,
	} from "flowbite-svelte-icons";
	import { page } from "$app/stores";
	import { browser } from '$app/environment';
	import { enhance } from '$app/forms';
    import { scale, fade } from 'svelte/transition';
    import image from '$lib/images/404.png';
	$: activeUrl = $page.url.pathname;

	let loaded = false;
	let notificationVisible = false;
	let notificationTitle;
	let notificationText;
	let messageBoxVisible = false;
	let orders = [];
	let popupInfo;
	var total = 0;

	var account_token;
	var orderid;
	var email;

	if (browser) {
		if (!window.localStorage.getItem('account_token')) {
			window.location.href = "/login";
		}

		fetch(`/orders?account_token=${window.localStorage.getItem('account_token')}`, {
				method: "GET"
			}).then(async function(response) {
				response = await response.json();
				
				if (response.status == "fail") {
					changeNotifState('Fail', "Your account is not registered. Please logout");
				}

				orders = response.result;
				loaded = true;
			})

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

	function getTotal(orderid) {
		total = 0;

		var itemsInOrder = JSON.parse(orders[orderid]['orderlist']);
		Object.values(itemsInOrder).forEach(item => {
			total += parseInt(item.price) * item.amount;
		});
	}

	function logout() {
		window.localStorage.removeItem('orders');
		window.localStorage.removeItem('account_token');
		window.location.href = "/login";
	}

	function changeNotifState(title, text) {
        notificationVisible = !notificationVisible;
        notificationText = text;
        notificationTitle = title;

        setTimeout(function() {
            notificationVisible = !notificationVisible
        },5000)
    }

	function revealOrder(event) {
		if (event.target.id == "arrow") {
			event.target.parentNode.parentNode.style.height = event.target.parentNode.parentNode.style.height == "150px"? event.target.parentNode.parentNode.style.height = "auto" : event.target.parentNode.parentNode.style.height = "150px";
		}
	}

	function changeMessageBoxState(mail, id) {
		orderid = id;
		email = mail; 
		account_token = window.localStorage.getItem('account_token');

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
					changeNotifState('Success', 'Removed order from list');
                }
                else if (result.data.status == "fail") {
					changeMessageBoxState();
                    changeNotifState('Fail', 'Could not remove item. Please try again later');
                }
                else if (result.data.status == "creds") {
					changeMessageBoxState();
                    changeNotifState('Fail', 'You account is not authorized to remove orders OR retry clicking the remove button');
                }
				else if (result.data.status == "invalid") {
					changeMessageBoxState();
					changeNotifState('Fail', 'Your account is not authorized to remove order. Try signing in again')
				}
            };
        }}>
		<div class="flex justify-center items-center flex-col fixed bottom-0 top-0 right-0 left-0 bg-black bg-opacity-90 z-[120]">
			<div class="relative bg-white w-[323px] h-[200px] rounded-md">
				<Input class="hidden" name='orderid' value={orderid}/>
				<Input class="hidden" name='email' value={email}/>
				<Input class="hidden" name='account_token' value={account_token}/>
				<h1 class="text-xl font-bold mx-5 mt-5">Are you sure?</h1>
				<h1 class="text-md font-normal text-gray-600 mx-5 mt-2">Are you sure you want to remove this this order from the list?</h1>
				<div class="absolute bottom-5 left-0 right-0 flex items-center justify-center mx-5">
					<Button outline=true on:click={changeMessageBoxState} class="mx-2 bottom-5 text-md">Cancel</Button>
					<Button type="submit" name='remove' class="mx-2 bottom-5 text-md">Remove</Button>
				</div>
			</div>
		</div>
	</form>
{/if}

<h1 class="text-2xl font-medium mx-8 mt-10">Orders</h1>
<h2 class="text-md text-gray-500 font-normal mx-8 mt-2">
	Complete or cancel orders
</h2>

{#if !loaded}
	<ListPlaceholder class="mt-10 mb-10 mx-8 space-y-10" />
{:else}
	{#if orders}
		<div class="grid mt-10 grid-rows-1 mx-8 gap-5">
			{#each Object.values(orders) as order, index}
				<form method="POST" action="?/remove" use:enhance={({ formElement, formData, action, cancel }) => {
					return async ({ result }) => {
						if (result.data.status == "success") {
							changeMessageBoxState();
							changeNotifState('Success', 'Removed order');
						}
						else if (result.data.status == "fail") {
							changeMessageBoxState();
							changeNotifState('Fail', 'Could not remove order. Please try again later');
						}
						else if (result.data.status == "creds") {
							changeMessageBoxState();
							changeNotifState('Fail', 'You account is not authorized to remove orders OR retry clicking the remove button');
						}
						else if (result.data.status == "invalid") {
							changeMessageBoxState();
							changeNotifState('Fail', 'Your account is not authorized to remove orders from the menu. Try signing in again')
						}
					};
				}}>
					<div class="overflow-hidden relative border-2 border-gray-300 rounded-lg h-[150px] w-full py-5 px-4">
						<h1 class="font-medium text-xl">{order.firstname}</h1>
						<h1 class="font-normal text-md text-gray-600">{order.email}</h1>
						<h1 class="font-normal text-md text-gray-600">{order.phone}</h1>
						<div class="flex items-center justify-center bottom-0 left-0 right-0 absolute bg-gradient-to-b from-transparent to-white">
							<AngleDownSolid on:click={() => getTotal(index)} id="arrow" class="mb-3 focus:outline-0 focus:border-0" on:click={revealOrder} color="#1A56DB"></AngleDownSolid>
						</div>
						<ul class="mt-8">
							{#each Object.values(JSON.parse(order['orderlist'])) as orderinfo}
								<li class="font-medium">x{orderinfo.amount} {orderinfo.name}</li>
							{/each}
						</ul>
						<div class="w-full border-t-2 mt-2 mb-2 border-gray-300"></div>
						<h1 class="text-lg font-normal">Total: R{total}</h1>

						<Button on:click={() => {changeMessageBoxState(order.email, order.orderid)}} class="focus:border-0 focus:outline-0 absolute top-3 right-3 text-lg font-bold rounded-full px-0 py-0 h-[30px] w-[30px]"><CloseCircleSolid class="focus:border-primary focus:outline-0 w-full h-full border-2 border-primary-50 rounded-full"></CloseCircleSolid></Button>
					</div>
				</form>
			{/each}
		</div>
	{:else}
		<div class="flex mt-10 justify-center items-center w-full">
			<img alt="404" src={image} class="w-[246px] h-[185px] opacity-40"/>
		</div>
	{/if}
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
