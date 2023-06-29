<script lang="ts">
  const alphabet = ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"];
  if (localStorage.getItem("kola") === null || localStorage.getItem("kola") === undefined || localStorage.getItem("kola") === "") {
    localStorage.setItem("kola", JSON.stringify([]));
  }
  if (localStorage.getItem("players") === null || localStorage.getItem("players") === undefined || localStorage.getItem("players") === "") {
    localStorage.setItem("players", JSON.stringify([]));
  }

  let localName = localStorage.getItem("name");
  let editName = "";
  let kola = JSON.parse(localStorage.getItem("kola"));

  function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  let player = "";
  let printable = false;
  let players = JSON.parse(localStorage.getItem("players"));

  const colors = ["", "lightgray", "lightgreen", "orange"];

  let today = new Date();
  const dd = String(today.getDate()).padStart(2, '0');
  const mm = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!
  const yyyy = today.getFullYear();

  let todayDate = `${dd}. ${mm}. ${yyyy}`;

  function clear() {
    players = [];
    kola = [];
    isSubmitted = false;
    localStorage.setItem("kola", "[]");
    localStorage.setItem("players", "[]");
    localStorage.setItem("name", "");
  }

  function isGoingOnward(name: string, prev_kolo: number) {
    let novo_kolo = kola[prev_kolo+1];
    //console.log(name, prev_kolo, novo_kolo);
    if (novo_kolo === undefined) return 0;
    for (let i = 0; i < novo_kolo.skupine.length; i++) {
      let skupina = novo_kolo.skupine[i];
      for (let n = 0; n < skupina.players.length; n++) {
        let player = skupina.players[n];
        if (name !== player.name) continue;
        if (player.notification !== undefined && player.notification.includes("z uporabo razmerij")) return 3;
        return 2;
      }
    }
    return 1;
  }

  function generateFirstKolo() {
    isSubmitted = false;
    shuffleArray(players);
    let skupine = [];
    for (let i = 0; i < Math.ceil(players.length/4); i++) {
      skupine.push({id: i+1, players: [
          {name: players[4*i], points: 0},
          {name: players[4*i+1], points: 0},
          {name: players[4*i+2], points: 0},
          {name: players[4*i+3], points: 0}
        ]});
    }
    localStorage.setItem("kola", JSON.stringify([
      {id: 1, skupine: skupine}
    ]));
    kola = JSON.parse(localStorage.getItem("kola"));
  }

  function calculateKolo() {
    isSubmitted = false;
    let kolo = kola[kola.length-1];
    let topPlayers1 = [];
    let topPlayers2 = [];
    let sidePlayers = [];
    for (let i = 0; i < kolo.skupine.length; i++) {
      let skupina = kolo.skupine[i];
      const top = skupina.players.sort((a,b) => b.points - a.points);
      const first = top[0];
      const second = top[1];
      const third = top[2];
      const razmerje = third.points/Math.abs(second.points);
      sidePlayers.push({name: third.name, ratio: razmerje});
      if (Math.round(Math.random()) === 0) {
        topPlayers1.push({name: first.name, points: 0});
        topPlayers2.push({name: second.name, points: 0});
      } else {
        topPlayers1.push({name: second.name, points: 0});
        topPlayers2.push({name: first.name, points: 0});
      }
    }
    console.log(sidePlayers);
    //console.log("pushed to arrays", topPlayers1, topPlayers2, sidePlayers);
    if ((topPlayers1.length + topPlayers2.length) % 4 !== 0) {
      const ratio = sidePlayers.sort((a,b) => b.ratio - a.ratio);
      const missingPlayers = (topPlayers1.length + topPlayers2.length) % 4;
      for (let i = 0; i < missingPlayers; i++) {
        if (topPlayers1.length % 4 !== 0) {
          topPlayers1.push({name: ratio[i].name, points: 0,
            notification: `Igralec je bil izbran za napredovanje z uporabo razmerij med 2. in 3. Dosegel je razmerje ${ratio[i].ratio}`});
          continue;
        }
        topPlayers2.push({name: ratio[i].name, points: 0,
          notification: `Igralec je bil izbran za napredovanje z uporabo razmerij med 2. in 3. Dosegel je razmerje ${ratio[i].ratio}`});
      }
    }
    shuffleArray(topPlayers1);
    shuffleArray(topPlayers2);
    let skupine = [];
    if (topPlayers1.length + topPlayers2.length <= 4) {
      let all = [...topPlayers2, ...topPlayers1];
      skupine.push({
        id: 1,
        players: all,
      })
    } else {
      let all = [...topPlayers1, ...topPlayers2];
      for (let i = 0; i < all.length/4; i++) {
        skupine.push({
          id: skupine.length + 1,
          players: [all[4 * i], all[4 * i + 1], all[4 * i + 2], all[4 * i + 3]]
        })
      }
    }
    kola.push({
      id: kolo.id+1,
      skupine: skupine,
    });
    kola = kola;
    localStorage.setItem("kola", JSON.stringify(kola));
    //console.log(kola);
  }

  function deleteKolo(kolo: number) {
    if (kola.length === 1) {
      localStorage.setItem("kola", "");
      kola = [];
      return;
    }
    kola.splice(kolo-1, 1);
    kola = kola;
    localStorage.setItem("kola", JSON.stringify(kola));
  }

  let endBest = undefined;

  let isSubmitted = kola.length > 1;

  function addPlayer() {
    players.push(player);
    localStorage.setItem("players", JSON.stringify(players));
    players = players;
    player = "";
  }

  function removePlayer(name: string) {
    const index = players.indexOf(name);
    if (index > -1) { // only splice array when item is found
      players.splice(index, 1); // 2nd parameter means remove one item only
    }
    localStorage.setItem("players", JSON.stringify(players));
    players = players;
  }

  function endCompetition() {
    const zadnjeKolo = kola.at(-1);
    const zadnjaSkupina = zadnjeKolo.skupine.at(-1);
    endBest = zadnjaSkupina.players.sort((a,b) => b.points - a.points);
  }
</script>

{#if printable}
  <h2>Rezultati tarok turnirja</h2>
  {#each kola as kolo, k}
    <div class="kolo">
      <h2>{kolo.id}. kolo</h2>
      <table>
        <tr>
          {#if kolo.skupine.length === 1}<th>Mesto</th>{:else}<th>Skupina</th>{/if}
          <th>Igralec</th>
          <th>Točke</th>
        </tr>
        {#each kolo.skupine as skupina, i}
          {#each skupina.players.sort((a,b) => b.points - a.points) as player, i}
            <tr>
              <td>{#if kolo.skupine.length !== 1}{skupina.id}{:else}{i+1}.{/if}</td>
              <td><span title="{player.notification}" style="color: {colors[isGoingOnward(player.name, k)]};">{player.name} {#if player.notification}⚠️{/if}</span></td>
              <td><span style="color: {player.points < 0 ? 'red' : (player.points > 0 ? 'green' : 'gray')}">{player.points}</span></td>
            </tr>
          {/each}
        {/each}
      </table>
    </div>
  {/each}
  <p/>
  Čestitamo prav vsem tekmovalcem in hvala za udeležbo.
  <p/>
  <div class="hiddenduringprint">
    <button on:click={() => printable = false}>Projekcijska oblika</button>
  </div>
{:else}
  <main>
    <h1>Tarok turnir
      {#if (localName === undefined || localName === null || localName === "")}
        <input bind:value={editName}>
        <button style="height: 30px; font-size: 13px;" on:click={() => {
          localStorage.setItem("name", editName);
          localName = localStorage.getItem("name");
        }}>Shrani ime turnirja</button>
      {:else}
        {localName}
      {/if}
      <span style="font-size: 16px;">(<span style="color: gray;">{todayDate}</span>)</span>
    </h1>

    {#if kola.length === 0}
      <h2>Igralci:</h2>
      <input bind:value={player} style="width: 90%;" type="text"><button on:click={addPlayer}>Dodaj</button>
      <p/>

      {#each players as playerVal}
        <div style="display:flex;justify-content:flex-end;align-items:center;">
          <span>{playerVal}</span>
          <div style="margin-left: auto;"><button on:click={() => removePlayer(playerVal)}>Odstrani</button></div>
        </div>
      {/each}

      <p/>
      {#if players.length >= 4}
        <button on:click={generateFirstKolo}>Generiraj prvo kolo</button>
      {/if}
    {/if}

    {#each kola as kolo, k}
      <h2>{kolo.id}. kolo <button style="height: 25px; font-size: 10px;" on:click={() => deleteKolo(kolo.id)}>Izbriši kolo</button></h2>

      {#if !isSubmitted}
        Postavitev po skupinah v tem kolu je neuradna. Še vedno lahko pride do sprememb.
      {/if}

      <div class="wrapper">
        {#each kolo.skupine as skupina, i}
          <div class="box {alphabet[i]}">
            <h3>{skupina.id}. skupina</h3>
            <table style="width: 100%;">
              <tr>
                <th style="text-align: left;">Igralec</th>
                <th style="text-align: right;">Točke</th>
              </tr>
              {#each skupina.players as player}
                <tr>
                  {#if !(player.name !== undefined && isSubmitted)}
                    <td><input type="text" bind:value={player.name} on:change={() => player.notification = "Igralec je bil vpisan ročno."}></td>
                    <td><input type="number" on:change={() => localStorage.setItem("kola", JSON.stringify(kola))} bind:value={player.points}></td>
                  {:else}
                    <td><span title="{player.notification}">
                      <span style="color: {colors[isGoingOnward(player.name, k)]}">{player.name}</span>
                      {#if player.notification}⚠️{/if}</span></td>
                    <td style="text-align: right; color: {player.points < 0 ? 'red' : (player.points > 0 ? 'green' : 'gray')}">{player.points}</td>
                  {/if}
                </tr>
              {/each}
            </table>
          </div>
        {/each}
      </div>
    {/each}
    <p/>
    {#if !isSubmitted && kola.length !== 0}
      <button on:click={() => isSubmitted = true}>Zaključi ročno vpisovanje imen</button>
      <p/>
    {/if}
    {#if kola.length !== 0 && kola.at(-1).skupine.length === 1 && isSubmitted}
      <button on:click={() => isSubmitted = false}>Uredi kola</button>
      <button on:click={endCompetition}>Zaključi turnir</button>
      <button on:click={clear}>Ustvari nov turnir</button>
    {:else}
      {#if kola.length !== 0 && isSubmitted}
        <button on:click={() => isSubmitted = false}>Uredi kola</button>
        <button on:click={calculateKolo}>Kalkuliraj novo kolo</button>
      {/if}
    {/if}
    <p/>
    {#if kola.length !== 0}
      <button on:click={() => printable = true}>Oblika za tisk</button>
    {/if}
    <p/>
    {#if kola.length !== 0 && endBest !== undefined}
      <h2>Zmagovalci</h2>
      <span style="font-size: 36px;">🥇 {endBest[0].name} s {endBest[0].points} točkami.</span><p/>
      <span style="font-size: 30px;">🥈 {endBest[1].name} s {endBest[1].points} točkami.</span><p/>
      <span style="font-size: 24px;">🥉 {endBest[2].name} s {endBest[2].points} točkami.</span><p/>
      <p/>
      Čestitamo prav vsem tekmovalcem in hvala za udeležbo.
    {/if}
  </main>
{/if}
