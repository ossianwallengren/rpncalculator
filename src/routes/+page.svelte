<script lang="ts">
    let stack: bigint[] = [];
    let current: bigint | null = null;

    let currentlyDown: Action | null = null;

    let buttons = {
        num: {
            sev: "7",
            eit: "8",
            nin: "9",
            for: "4",
            fiv: "5",
            six: "6",
            one: "1",
            two: "2",
            tre: "3",
            zer: "0",
            dot: ".",
        },

        mv: {
            bsp: "<",
            clr: "AC",
            psh: "v",
        },

        op: {
            div: "/",
            mul: "*",
            sub: "-",
            add: "+",
        },
    };

    type InnerKeys<T> = T extends object ? keyof T : never;
    type Action = InnerKeys<(typeof buttons)[keyof typeof buttons]>;
    type ActionIn<T extends keyof typeof buttons> = keyof (typeof buttons)[T];

    function asAction(a: string): Action {
        if (Object.values(buttons).flatMap(Object.keys).includes(a))
            return a as Action;
        throw new Error(`${a} is not a valid action`);
    }

    let codeKeymap = new Map<string, Action>();
    for (let [id, d] of Object.entries(buttons.num)) {
        if (id === "dot") continue;
        codeKeymap.set(`Digit${d}`, id as ActionIn<"num">);
        codeKeymap.set(`Numpad${d}`, id as ActionIn<"num">);
    }
    let keyKeymap = new Map<string, Action>();
    keyKeymap.set("Backspace", "bsp");
    keyKeymap.set("Escape", "clr");
    keyKeymap.set(" ", "psh");
    keyKeymap.set("Enter", "psh");
    for (let [id, op] of Object.entries(buttons.op)) {
        keyKeymap.set(op, id as ActionIn<"num">);
    }

    window.onkeydown = (event) => {
        let t = keyKeymap.get(event.key) ?? codeKeymap.get(event.code);
        if (t === undefined) return;
        action(t);
        currentlyDown = t;
    };

    window.onkeyup = () => {
        currentlyDown = null;
    };

    function action(a: Action) {
        if (a in buttons.num) {
            let x = buttons.num[a as keyof typeof buttons.num];
            current = (current ?? 0n) * 10n + BigInt(x);
        } else if (a === "bsp") {
            if (current === null) {
                current = stack.at(-1) ?? null;
                stack = [...stack.slice(0, stack.length - 1)];
            } else {
                current = (current ?? 0n) / 10n;
                if (current === 0n) current = null;
            }
        } else if (a === "psh") {
            if (current !== null) stack = [...stack, current ?? 0];
            current = null;
        } else if (a === "clr") {
            current = null;
        } else if (a === "add" || a === "sub" || a === "mul" || a === "div") {
            binop(a);
        }
    }

    function binop(op: ActionIn<"op">) {
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
            case "add":
                current = lhs + rhs;
                break;
            case "sub":
                current = lhs - rhs;
                break;
            case "mul":
                current = lhs * rhs;
                break;
            case "div":
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

    {#each Object.entries(buttons) as [class_, btns]}
        {#each Object.entries(btns) as [id, content]}
            <button
                class:active={currentlyDown === id}
                on:click={() => action(asAction(id))}
                style={`grid-area: ${id}`}
                class={class_}>{content}</button
            >
        {/each}
    {/each}
</div>

<style>
    button {
        border: none;
    }

    button:active,
    button.active {
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
        button:active,
        button.active {
            filter: brightness(150%);
        }

        .mv {
            background-color: #a73825;
        }

        .op {
            background-color: #452fb3;
        }

        .num,
        .fill {
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
