<script lang="ts">
    let stack: number[] = [];
    let current: number | null = null;

    function click(this: HTMLButtonElement) {
        let t = this.textContent;
        if (!t) return;
        if ("0123456789".includes(t)) {
            current = (current ?? 0) * 10 + +t;
        } else if (t === "<") {
            current = Math.floor((current ?? 0) / 10);
            if (current === 0) current = null;
        } else if (t === "v") {
            if (current !== null) stack = [...stack, current ?? 0];
            current = null;
        } else if (t === "AC") {
            current = null;
        } else if (t === "+" || t === "-" || t === "*" || t === "/") {
            binop(t);
        }
    }

    function binop(op: "+" | "-" | "*" | "/") {
        if (current === null) {
            let val = stack.at(-1);
            if (!val) {
                return;
            }
            current = val;
            stack = stack.slice(0, stack.length - 1);
        }
        let lhs = stack.at(-1);
        if (!lhs) return;
        stack = stack.slice(0, stack.length - 1);
        let rhs = current;
        switch (op) {
            case "+":
                current = lhs + rhs;
                break;
            case "-":
                current = lhs - rhs;
                break;
            case "*":
                current = lhs * rhs;
                break;
            case "/":
                current = lhs / rhs;
                break;
        }
    }
</script>

<div class="container">
    <div style="grid-area: val;" class="val">
        <div class="stack">
            {#each stack as val}
                <p>{val}</p>
            {/each}
        </div>
        <p class="current">{current ?? ""}</p>
    </div>

    <div style="grid-area: fil" class="fill"></div>

    <button on:click={click} style="grid-area: sev" class="num">7</button>
    <button on:click={click} style="grid-area: eit" class="num">8</button>
    <button on:click={click} style="grid-area: nin" class="num">9</button>
    <button on:click={click} style="grid-area: for" class="num">4</button>
    <button on:click={click} style="grid-area: fiv" class="num">5</button>
    <button on:click={click} style="grid-area: six" class="num">6</button>
    <button on:click={click} style="grid-area: one" class="num">1</button>
    <button on:click={click} style="grid-area: two" class="num">2</button>
    <button on:click={click} style="grid-area: tre" class="num">3</button>
    <button on:click={click} style="grid-area: zer" class="num">0</button>
    <button on:click={click} style="grid-area: dot" class="num">.</button>

    <button on:click={click} style="grid-area: bsp" class="mv">&lt;</button>
    <button on:click={click} style="grid-area: clr" class="mv">AC</button>
    <button on:click={click} style="grid-area: psh" class="mv">v</button>

    <button on:click={click} style="grid-area: div" class="op">/</button>
    <button on:click={click} style="grid-area: mul" class="op">*</button>
    <button on:click={click} style="grid-area: sub" class="op">-</button>
    <button on:click={click} style="grid-area: add" class="op">+</button>
</div>

<style>
    button {
        border: none;
    }

    button:active {
        filter: brightness(80%);
    }

    .mv {
        background-color: #da8070;
    }

    .op {
        background-color: #a090ef;
    }

    .num {
        background-color: #f6f6f6;
    }

    .val {
        background-color: #ffffff;
        display: flex;
        align-items: start;
        flex-direction: column;
        overflow-x: auto;
        padding: 0.5em;
    }

    .stack {
        display: flex;
        gap: 1em;
    }

    .current {
        font-size: 48px;
    }

    .fill {
        background-color: #f6f6f6;
    }

    .container {
        min-height: 100vh;
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: repeat(6, 1fr);
        grid-template-areas:
            "val val val val"
            "bsp clr fil fil"
            "sev eit nin div"
            "for fiv six mul"
            "one two tre sub"
            "zer dot psh add";
        gap: 1px;
        background-color: #545454;
    }

    @media (prefers-color-scheme: dark) {
        button:active {
            filter: brightness(150%);
        }

        .mv {
            background-color: #a73825;
        }

        .op {
            background-color: #452fb3;
        }

        .num, .fill {
            background-color: #444;
        }

        .val {
            background-color: #222;
        }

        .container {
            background-color: #777;
        }
    }
</style>
