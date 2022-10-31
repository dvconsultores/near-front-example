<template>
	<v-container>
		<v-row class="text-center">
			<v-col cols="12">
				<v-img
					:src="require('../assets/logo.png')"
					class="my-2"
					contain
					height="200"
				/>
			</v-col>

			<v-col class="mb-4">
				<h1 class="display-2 font-weight-bold mb-2">
					Welcome to Near Protocol Example
				</h1>
			</v-col>
		</v-row>
    <v-row class="text-center">
			<v-col class="mb-4">
				<v-btn @click="login" color="success" :loading="loading1">{{ wallet_name }}</v-btn>
			</v-col>

      <v-col class="mb-4">
				<v-btn @click="logout" color="error">Logout</v-btn>
			</v-col>
		</v-row>
		<v-row class="text-center">
			<v-col class="mb-4">
				<v-btn
					@click="get_get_sum_of_deposits"
					color="primary"
					:loading="loading2"
					>Get Deposit Sum</v-btn
				>
				<h3>{{ sum_values }}</h3>
			</v-col>

			<v-col class="mb-4">
				<v-btn @click="deposit_to_storage" color="primary" :loading="loading3"
					>Deposit Storage</v-btn
				>
			</v-col>
		</v-row>
	</v-container>
</template>

<script>
	import { Buffer } from "buffer";
	import * as nearAPI from "near-api-js";
	import { CONFIG } from "@/services/api";
	const { connect, keyStores, WalletConnection, Contract } = nearAPI;
	const CONTRACT_NAME = "dev-1666979301533-26368053949992";
	// @ts-ignore
	window.Buffer = Buffer;
	export default {
		name: "HellNear",

		data: () => ({
			whatsNext: [],
			loading1: false,
			loading2: false,
			loading3: false,
			sum_values: "",
			wallet_name: localStorage.getItem('user'),
		}),
		mounted() {
			this.get_get_sum_of_deposits();
		},
		methods: {
			async login() {
        this.loading1 = true;
				// connect to NEAR
				const near = await connect(
					CONFIG(new keyStores.BrowserLocalStorageKeyStore())
				);
				const wallet = new WalletConnection(near);
				// wallet.requestSignIn(CONTRACT_NAME);
        wallet.requestSignIn({
					contractId: CONTRACT_NAME,
					methodNames: [], // optional
					successUrl: "http://localhost:8080/", // optional redirect URL on success
					failureUrl: "http://localhost:8080/", // optional redirect URL on failure
				});
				
			},
      async logout() {
				// connect to NEAR
				const near = await connect(
					CONFIG(new keyStores.BrowserLocalStorageKeyStore())
				);
				const wallet = new WalletConnection(near);
				// wallet.requestSignIn(CONTRACT_NAME);
				wallet.signOut();
        localStorage.setItem('user', "LOGIN");
        window.location = "http://localhost:8080/"
			},
			async get_get_sum_of_deposits() {
				this.loading2 = true;
				const near = await connect(
					CONFIG(new keyStores.BrowserLocalStorageKeyStore())
				);
				// create wallet connection
				// const account = await near.account();
				const wallet = new WalletConnection(near);
				// console.log(near);
				const contract = new Contract(wallet.account(), CONTRACT_NAME, {
					viewMethods: ["get_sum_of_deposits"],
					changeMethods: [],
					sender: wallet.account(),
				});
				if (wallet.isSignedIn()) {
					this.sum_values = await contract.get_sum_of_deposits() / 1000000000000000000000000;
					console.log(this.sum_values);
				}
				this.loading2 = false;
			},
			async deposit_to_storage() {
        this.get_user();
				this.loading3 = true;
				const near = await connect(
					CONFIG(new keyStores.BrowserLocalStorageKeyStore())
				);
				// create wallet connection
				// const account = await near.account();
				const wallet = new WalletConnection(near);
				// console.log(near);
				const contract = new Contract(wallet.account(), CONTRACT_NAME, {
					viewMethods: [],
					changeMethods: ["deposit_to_storage"],
					sender: wallet.account(),
				});
				if (wallet.isSignedIn()) {
					await contract.deposit_to_storage(
						{},
						"300000000000000", // attached GAS (optional)
						"1000000000000000000000001" // attached deposit in yoctoNEAR (optional)
					);
				}
				this.loading3 = false;
			},
			async get_user() {
				const near = await connect(
					CONFIG(new keyStores.BrowserLocalStorageKeyStore())
				);
				// create wallet connection
				// const account = await near.account();
				const wallet = new WalletConnection(near);
				localStorage.setItem("user", wallet.getAccountId());
			},
		},
	};
</script>
