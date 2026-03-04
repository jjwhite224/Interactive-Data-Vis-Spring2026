# Jelani White - Interactive-Data-Vis-Spring2026

## Lab 0

<img src="https://upload.wikimedia.org/wikipedia/en/0/0c/Liverpool_FC.svg" width="200">

<br> <table> <tr> <th>Season</th> <th>Wins</th> </tr> <tr> <td>24/25</td> <td>26</td> </tr> <tr> <td>25/26</td> <td>14</td> </tr> </table> 

<h3>My Favorite Players</h3>

<ul id="player-list">
<li>Szoboszlai</li>
<li>Ekitike</li>
<li>Van Dijk</li>
</ul>
<input id="player-input" placeholder="Add a player" ></input>

```js

const input = document.querySelector("#player-input")
const list = document.querySelector("#player-list")

input.addEventListener("keydown", (event) => {

  if (event.key === "Enter") {

    const name = input.value.trim()
    if (!name) return

    const li = document.createElement("li")
    li.textContent = name

    list.appendChild(li)

    input.value = ""

  }

})

```

