<script>
    import { each } from "svelte/internal";

    let vehicle = {};
    let vin = "";
    $: hasResults = false;
    const searchVIN = async () => {
        // https://vpic.nhtsa.dot.gov/api/vehicles/decodevinvalues/1FTFW1RG2LFB85006?format=json
        const url = `https://vpic.nhtsa.dot.gov/api/vehicles/decodevinvalues/${vin}?format=json`;
        const res = await fetch(url);
        const json = await res.json();
        const temp = {};
        Object.entries(json.Results[0]).forEach(
            (it) =>
                it[1].length &&
                it[1].toLowerCase() != "not applicable" &&
                (temp[it[0]] = it[1])
        );

        const {
            MakeID,
            Make,
            ModelID,
            Model,
            ModelYear,
            Trim,
            VIN,
            VehicleType,
            ...additionalData
        } = temp;

        vehicle = {
            MakeID,
            Make,
            ModelID,
            Model,
            ModelYear,
            Trim,
            VIN,
            VehicleType,
            additionalData,
        };

        hasResults = true;
        console.log(vehicle);
    };
</script>

<main>
    <div class="container">
        <form on:submit|preventDefault={searchVIN} method="POST">
            <div class="row">
                <div class="col-10">
                    <div class="form-floating mb-3">
                        <input
                            bind:value={vin}
                            type="text"
                            class="form-control"
                            id="floatingInput"
                            placeholder="123abc"
                        />
                        <label for="floatingInput">VIN Number</label>
                    </div>
                </div>
                <div class="col-2">
                    <button class="btn btn-primary" type="submit">Search</button
                    >
                </div>
            </div>
        </form>
    </div>

    {#if hasResults}
        <div class="container">
            <div class="card" style="width: 18rem;">
                <div class="card-header">
                    {vehicle.ModelYear}
                    {vehicle.Make}
                    {vehicle.Model}
                </div>
                <ul class="list-group list-group-flush">
                    {#each Object.entries(vehicle.additionalData) as item (`${item[0]}_${item[1]}`)}
                        <li class="list-group-item">{item[0]}: {item[1]}</li>
                    {/each}
                </ul>
            </div>
        </div>
    {/if}
</main>

<style>
    main {
        padding: 50px;
    }
</style>
