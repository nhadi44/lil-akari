<script lang="ts">
  import { onMount } from "svelte";
  import { Connection } from "@solana/web3.js";
  import { Provider, web3 } from "@project-serum/anchor";
  import { fade } from "svelte/transition";
  import Button from "./components/Button.svelte";
  import Header from "./components/CardHeader.svelte";

  import { candyMachineState, userState } from "./lib/store";
  import {
    getCandyMachineState,
    checkWalletConnected,
    getUserBalance,
    existsOwnerSPLToken,
  } from "./lib/state-helpers";

  /***********************************/
  // Customise the app by changing the following variables.
  const TITLE = "555 LIL AKARI";
  const DESCRTIPTION =
    "is a collection of unique characters from Japanese philosophy that will create a virtuous community on the #solana blockchain";
  const HEADER_TITLE = "lilakari";
  const HEADER_LINK = "/";
  // Your image or GIF needs to be in the /public folder for this to work
  const IMAGE_LINK = "/logo.gif";
  /***********************************/

  let { solana } = window as any;
  const rpcUrl = import.meta.env.VITE_APP_SOLANA_RPC_HOST?.toString();
  const cluster = import.meta.env.VITE_APP_SOLANA_NETWORK?.toString();
  const candyMachineId = import.meta.env.VITE_APP_CANDY_MACHINE_ID?.toString();
  const opts = { preflightCommitment: "processed" };

  let siteLoading = true;
  let errorOcurred = false;
  let connection: Connection;
  let provider: Provider;
  let candyMachinePublicKey: web3.PublicKey;

  $: itemsRedeemed = $candyMachineState?.state.itemsRedeemed;
  $: itemsAvailable = $candyMachineState?.state.itemsAvailable;

  function checkEnvironmentVariables() {
    // Check if populated
    if (!rpcUrl || !candyMachineId || !cluster) {
      if (!rpcUrl) {
        console.error("RPC URL not populated");
      }
      if (!candyMachineId) {
        console.error("Candy Machine ID not populated");
      }
      if (!cluster) {
        console.error("Environment not populated");
      }
      return true;
    }
    if (candyMachineId.length < 32 || candyMachineId.length > 44) {
      console.error(
        "Candy Machine Public Key is invalid. Enter a length in-between 32 and 44 characters"
      );
      return true;
    }
    return false;
  }

  onMount(async () => {
    solana = (window as any).solana;
    // Check if environement variables are populated
    errorOcurred = checkEnvironmentVariables();
    if (errorOcurred) {
      return;
    }

    // If env variables populated, create provider, PK and connection
    connection = new Connection(rpcUrl);
    provider = new Provider(
      connection,
      solana,
      opts.preflightCommitment as web3.ConfirmOptions
    );
    candyMachinePublicKey = new web3.PublicKey(candyMachineId);

    // Get candy machine state
    $candyMachineState = await getCandyMachineState(
      candyMachinePublicKey,
      provider
    );
    // Establish connection to wallet
    if (solana?.isPhantom) {
      $userState.walletPublicKey = await checkWalletConnected(solana);
      if ($userState.walletPublicKey) {
        // Get User Balance
        $userState.userBalance = await getUserBalance(
          $userState.walletPublicKey,
          connection
        );
        // If whitelist config populated, check if user is whitelisted (ie. check if they have token)
        if ($candyMachineState.state.whitelistMintSettings) {
          $userState.isWhiteListed = await existsOwnerSPLToken(
            $userState.walletPublicKey,
            connection,
            $candyMachineState.state.whitelistMintSettings?.mint
          );
        }
      }
    }

    // Stop loading
    siteLoading = false;
  });
</script>

<main class="h-screen bg-[url(/bg.jpg)] bg-no-repeat bg-cover">
  <!-- Error section -->
  {#if errorOcurred}
    <div class="h-full flex">
      <div class="m-auto">
        An error occurred. Please check if your environment variables have been
        populated correctly and redeploy the applcation.
      </div>
    </div>
    <!-- Loading Section -->
  {:else if siteLoading && !errorOcurred}
    <div class=" h-full flex">
      <div class="lds-hourglass m-auto" />
    </div>
  {:else}
    <!-- Menu Bar -->
    {#if HEADER_TITLE}
      <div
        class="flex justify-between text-slate-100 fixed w-full px-6 py-1 xl:py-2 left-0 top-0 items-center z-10"
      >
        <nav>
          <a href={HEADER_LINK} class="uppercase text-2xl xl:text-3xl"
            >{HEADER_TITLE}</a
          >
        </nav>
        <ul class="flex space-x-5">
          <li
            class="capitalize bg-slate-700 px-3 py-1 rounded-md  tracking-widest text-[12px] xl:text-[20px] hover:bg-slate-800 cursor-pointer"
          >
            <a href="#roadmap">roadmap</a>
          </li>
          <li
            class="capitalize bg-slate-700 px-3 py-1 rounded-md tracking-widest text-[12px] xl:text-[20px] hover:bg-slate-800 cursor-pointer"
          >
            <a href="#team">team</a>
          </li>
        </ul>
      </div>
    {/if}
    <!-- Card -->
    <div class="flex items-center justify-center min-h-screen">
      <div
        class="max-w-sm 2xl:max-w-xl bg-stone-500 rounded-lg border-2 "
        transition:fade
      >
        <!-- Top Bar -->
        <Header />
        <hr />
        <br />
        <!-- Main Body -->
        <div class="px-5 py-4">
          <div class="text-slate-100">
            <p class="capitalize mb-3">mint date</p>
            <p class="capitalize">
              Whitelist (17:50 UTC) || Public (18:00 UTC)
            </p>
          </div>
          <img src={IMAGE_LINK} alt="" class=" w-1/2 mx-auto m-5 rounded-sm" />
          <div
            class=" text-lg sm:text-2xl font-mono font-bold py-5 tracking-widest"
          >
            {TITLE}
          </div>
          <div class="text-sm sm:text-md font-semibold pb-5 text-gray-200 ">
            {DESCRTIPTION}
          </div>
          <Button {connection} />

          <div class=" tracking-widest font-bold text-sm pt-3 text-gray-700">
            {itemsRedeemed}/{itemsAvailable} claimed
          </div>
          <div class="flex flex-col pt-3">
            {#if $userState.solanaExplorerLink}
              <a
                href={$userState.solanaExplorerLink}
                target="_blank"
                class="text-purple-700 font-semibold  p-1"
                >View on Solana Explorer</a
              >
            {/if}
          </div>
        </div>
      </div>
    </div>
  {/if}
</main>
<div class="min-h-screen bg-[url(/bg.jpg)] bg-no-repeat bg-cover " id="roadmap">
  <div class="relative w-full min-h-screen">
    <div class="px-6 py-10 2xl:py-16 flex items-center justify-center">
      <h1 class="text-white uppercase font-semibold text-3xl">Roadmap</h1>
    </div>
    <div
      class="absolute w-full top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-[43%]"
    >
      <div class="bg-slate-100 flex space-x-8 px-3 justify-between">
        {#each { length: 11 } as _, i}
          <p class="text-xl uppercase font-semibold">{"lil akari"}</p>
        {/each}
      </div>
      <div
        class="bg-gray-800 w-full 2xl:h-[40rem] overflow-hidden text-white px-3 py-3 2xl:py-10 bg-opacity-[95%]"
      >
        <div class="grid grid-cols-3 gap-7">
          <div class="">
            <h1 class="mb-3 text-2xl 2xl:text-3xl capitalize font-semibold">
              phase 1
            </h1>
            <div class="2xl:text-2xl">
              <p class="mb-3">
                Building Community first we will build a community on twitter,
                then we will build on discord. until the art is owned by all our
                buyers with the LIL DAO role
              </p>
              <p class="mb-3">
                Community Impact Fund 20% of mint sales go to the Impact Fund
                and 80% of artist royalties go to the impact fund. All
                transactions are carried out through public wallets. Interactive
                dashboard & voting mechanism deployed. Lil Akari NFT holders
                discuss, vote & reap rewards from proposed sustainability-led
                projects
              </p>
              <p class="mb-3">
                Auction Ten 1/1 Lil Akari Auctioned progressively once sold out
                to LIL holders. 50% of the proceeds goes to the Impact Fund and
                25% goes to the marketing wallet to develop the project
                post-launch.
              </p>
            </div>
          </div>
          <div class="">
            <h1 class="mb-3 text-2xl xl:text-3xl capitalize font-semibold">
              phase 2
            </h1>
            <div class="2xl:text-2xl">
              <p class="mb-3">UTILITY & REWARD</p>
              <p class="mb-3">
                $LIL Staking Stake your LIL AKARI NFT for $LIL, providing
                utility in the Gen2 collection, impact investment governance and
                royalties. Find out more about $LIL utility.
              </p>
              <p class="mb-3">
                Gen 2 Collection | AKARI PUNK LIL akari is the universe in the
                present, and we will make LIL akari in the future. we will bring
                LIL akari to the metaverse
              </p>
              <p class="mb-3">
                DAO AKARI there will be weekly giveaways for holders, project
                collaboration whitelists and contributing to take steps lil
                akari
              </p>
            </div>
          </div>
          <div class="">
            <h1 class="mb-3 text-2xl xl:text-3xl capitalize font-semibold">
              phase 3
            </h1>
            <div class="2xl:text-2xl">
              <p class="mb-3">
                Long Term Value Holdler Giveaways Hosting regular competitions,
                LIL Akari events for LIL holders with SOL prizes & art
                giveaways.
              </p>
              <p class="mb-3">
                Sustainable Merch Drop Sustainable hoodies, kimonos & shoes
                designed by one of our team of experienced design artists.
                Secret codes embedded in certain items, available at discounted
                prices via Solana or $LIL.
              </p>
              <p class="mb-3">
                Akari Meetup we are going to make an event in one of the
                beautiful countries. we can relax on the beach camping in the
                mountains and the NFT holder will decide the place. and it will
                continue
              </p>
            </div>
          </div>
        </div>
      </div>
      <div class="bg-slate-100 flex space-x-8 px-3 justify-between">
        {#each { length: 11 } as _, i}
          <p class="text-xl uppercase font-semibold">{"lil akari"}</p>
        {/each}
      </div>
    </div>
  </div>
</div>
<div
  class="min-h-screen bg-[url(/bg.jpg)] bg-no-repeat bg-cover relative"
  id="team"
>
  <div class="px-6 py-20 2xl:py-36 flex justify-center">
    <h1 class="text-slate-100 uppercase text-3xl 2xl:text-5xl font-semibold">
      team
    </h1>
  </div>
  <div
    class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 w-full"
  >
    <div class="grid grid-cols-4 gap-5 px-6">
      <div
        class="w-full 2xl:w-4/5 h-80 2xl:h-96 bg-[url(/1.png)] bg-contain 2xl:bg-cover bg-no-repeat relative rouned-sm "
      >
        <div
          class="absolute -bottom-14 2xl:-bottom-20 text-center flex justify-center w-full flex-col bg-gray-800 rounded-sm py-2"
        >
          <h1 class="text-slate-100 font-semibold uppercase text-2xl">Hisao</h1>
          <h1 class="text-white capitalize">Project Director</h1>
        </div>
      </div>
      <div
        class="w-full 2xl:w-4/5 h-80 2xl:h-96 bg-[url(/2.png)] bg-contain 2xl:bg-cover bg-no-repeat relative rouned-sm "
      >
        <div
          class="absolute -bottom-14 2xl:-bottom-20 text-center flex justify-center w-full flex-col bg-gray-800 rounded-sm py-2"
        >
          <h1 class="text-slate-100 font-semibold uppercase text-2xl">
            Satoshi
          </h1>
          <h1 class="text-white capitalize">art Director</h1>
        </div>
      </div>
      <div
        class="w-full 2xl:w-4/5 h-80 2xl:h-96 bg-[url(/3.png)] bg-contain 2xl:bg-cover bg-no-repeat relative rouned-sm "
      >
        <div
          class="absolute -bottom-14 2xl:-bottom-20 text-center flex justify-center w-full flex-col bg-gray-800 rounded-sm py-2"
        >
          <h1 class="text-slate-100 font-semibold uppercase text-2xl">
            Kenshin
          </h1>
          <h1 class="text-white capitalize">community manager</h1>
        </div>
      </div>
      <div
        class="w-full 2xl:w-4/5 h-80 2xl:h-96 bg-[url(/4.png)] bg-contain 2xl:bg-cover bg-no-repeat relative rouned-sm "
      >
        <div
          class="absolute -bottom-14 2xl:-bottom-20 text-center flex justify-center w-full flex-col bg-gray-800 rounded-sm py-2"
        >
          <h1 class="text-slate-100 font-semibold uppercase text-2xl">Kira</h1>
          <h1 class="text-white capitalize">Developer</h1>
        </div>
      </div>
    </div>
  </div>
</div>
