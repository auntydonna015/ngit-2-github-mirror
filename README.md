# ngit to GitHub Mirror Walkthrough

Quick cheat-sheet:

* [ngit](https://ngit.dev): the command-line tool that connects Git to nostr
* [Git Relays Authorized via Signed-Nostr Proofs (GRASP)](https://gitworkshop.dev/npub15qydau2hjma6ngxkl2cyar74wzyjshvl65za5k5rl69264ar2exs5cyejr/relay.ngit.dev/grasp): hosts the repositories
* [GitWorkshop.dev](https://gitworkshop.dev/): the first web client
* Git: stores the code, every version of it
* Nostr: carries the signatures, issues, PRs, and comments

Now let's go.

## Tutorial

Ok, so first things first. Get git and curl on your terminal (or just Git Bash on Windows, it comes with both) and make sure you're inside the folder of the repository that you want to mirror.

**1. Install ngit.**

[The installer](https://ngit.dev/install.sh) will verify that it's downloaded properly, then once that's done, it will add `ngit` and `git-remote-nostr`, [which teaches Git to speak](https://ngit.dev/how-it-works) `nostr://` URLs:

```
curl -fsSL https://ngit.dev/install.sh | bash
```

You know it worked when you see: "Installed ngit v3.0.1". If it also warns that ngit isn't on your PATH, run the `export PATH=...` line it prints, and add that same line to your `~/.zshrc` so it sticks.

**2. Get your nostr remote signer.**

First off, I don't know who needs to hear this but: never paste your nsec anywhere! Use [nak](https://github.com/fiatjaf/nak) here as it's perfect for working on the terminal. Install it with its one-liner:

```
curl -sSL https://raw.githubusercontent.com/fiatjaf/nak/master/install.sh | sh
```

Then lock your key behind a password. This asks for your nsec and a new password without showing them or saving them in your shell history, and saves the result as `you.ncryptsec` in your home folder, outside your repository:

```
read -s "NSEC?your nsec: "; echo; read -s "PW?new password: "; echo; nak key encrypt "$NSEC" "$PW" > ~/you.ncryptsec; unset NSEC PW
```

After that you can start a bunker, and just make sure that you leave it running, step 3 needs it:

```
nak bunker --sec "$(cat ~/you.ncryptsec)" --profile you wss://nos.lol
```

It worked when nak prints a `bunker://` address. Make sure to check that the printed npub is yours.

**3. Sign this repository in from a second terminal, again inside your project's folder.**

If you're paranoid, don't ever use `--nsec`, because otherwise your private key gets written into your shell history in plain text. That's the nice thing with nak's bunker: your key never leaves that first terminal and ngit only ever receives signatures:

```
ngit account login --local --bunker-url 'bunker://...'
```

Paste the full address in place of `bunker://...`, keeping the single quotes.

It worked when ngit answers: "logged in to this local repository as you".

**4. [Announce the repository](https://ngit.dev/quickstart)**

Keeping GitHub among its servers is what makes it a mirror.

```
ngit init --name your-project --additional-clone https://github.com/you/your-project.git -g relay.ngit.dev -g gitnostr.com -d
```

Swap in your GitHub URL, the `https://github.com/<user>/<repo>.git` address from GitHub's Code button, and your project's name.

`--additional-clone` is what keeps GitHub in the loop.

`-g` adds [free community servers](https://ngit.dev/grasp/).

`-d` accepts the defaults.

It worked when you see "share your repository:". A couple of relays failing along the way is normal. ngit pushes your code to the new servers right away and repoints `origin` at the nostr URL (your old GitHub remote is still there, now called `github`).

Now, one ordinary `git push` feeds both GitHub and the nostr servers.

Thank you for reading this far, please consider mirroring one of your repositories today and [donating to OpenSats](https://opensats.org/donate).

You can find Dan on nostr as [DanConwayDev](https://njump.me/npub15qydau2hjma6ngxkl2cyar74wzyjshvl65za5k5rl69264ar2exs5cyejr) and his repositories on [gitworkshop.dev](https://gitworkshop.dev/danconwaydev.com).
