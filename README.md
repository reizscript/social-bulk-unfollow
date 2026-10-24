# social-bulk-unfollow

Browser-console scripts that bulk-unfollow accounts from your **Instagram** or **TikTok** Following list. Each script auto-scrolls the list to load more accounts and keeps going until nothing new loads.

> ⚠️ **Use at your own risk.** Automating actions on social platforms may violate their Terms of Use and can lead to action blocks or account restrictions. See the [Disclaimer](#disclaimer).

## Contents

- [Instagram](#instagram)
- [TikTok](#tiktok)
- [Tips](#tips)
- [License](#license)
- [Disclaimer](#disclaimer)
- [Contributing](#contributing)

## Instagram

**Features**

- Auto-scrolls the Following list to load more accounts
- Handles both "Following" and "Requested" buttons
- Randomized 3-6 second delays, plus a 60-second break every 20 unfollows
- Stops after 3 scrolls that load nothing new

**Usage**

1. Log in to Instagram on desktop and open your profile.
2. Click **Following** to open the list dialog.
3. Open DevTools (`F12` or `Ctrl/Cmd + Shift + J`) and go to the **Console** tab.
4. Paste the script below and press Enter.
5. Watch the console for progress (`Unfollowed 1`, `Unfollowed 2`, ...).

```js
(async () => {
  const delay = (ms) => new Promise((r) => setTimeout(r, ms));

  // Find the scrollable list inside the dialog
  const dialog = document.querySelector('div[role="dialog"]');
  const scroller = [...dialog.querySelectorAll('div')].find(
    (el) => el.scrollHeight > el.clientHeight + 20 &&
            getComputedStyle(el).overflowY !== 'visible'
  );

  let emptyRounds = 0;
  let count = 0;

  while (emptyRounds < 3) {
    const buttons = [...dialog.querySelectorAll('button')].filter(
      (b) => ['Following', 'Requested'].includes(b.innerText.trim())
    );

    if (buttons.length === 0) {
      // nothing to unfollow right now: scroll to load more
      const before = scroller.scrollHeight;
      scroller.scrollTop = scroller.scrollHeight;
      await delay(2500);
      if (scroller.scrollHeight === before) emptyRounds++;
      else emptyRounds = 0;
      continue;
    }

    emptyRounds = 0;
    const button = buttons[0];
    button.click();
    await delay(800);

    const confirm = [...document.querySelectorAll('button')].find(
      (b) => b.innerText.trim() === 'Unfollow'
    );
    if (confirm) confirm.click();

    count++;
    console.log(`Unfollowed ${count}`);

    // random delay of 3-6s to look less bot-like
    await delay(3000 + Math.random() * 3000);

    // pause longer every 20 unfollows
    if (count % 20 === 0) await delay(60000);
  }

  console.log('Done');
})();
```

## TikTok

**Features**

- Matches both "Following" and "Friends" (mutual follow) buttons
- Skips the Following / Followers / Friends tabs so they are never clicked by mistake
- Remembers clicked buttons, so a failed unfollow can't loop forever
- Finds the scrollable list automatically, with no hardcoded selectors
- Randomized 4-8 second delays, plus a 90-second break every 15 unfollows

**Usage**

1. Log in to TikTok on desktop and open your profile.
2. Click **Following** to open the list popup.
3. Open DevTools (`F12` or `Ctrl/Cmd + Shift + J`) and go to the **Console** tab.
4. Paste the script below and press Enter.

> Works with the English interface only, because it matches button text.

```js
(async () => {
  const delay = (ms) => new Promise((r) => setTimeout(r, ms));
  const LABELS = ['Following', 'Friends']; // mutual follows show "Friends"
  const clicked = new WeakSet();

  // A real list-row button: not a tab, and sits next to a profile link
  const isUserRow = (b) => {
    if (b.closest('[role="tab"], [role="tablist"]')) return false;
    let el = b.parentElement;
    for (let i = 0; i < 3 && el; i++, el = el.parentElement) {
      if (el.querySelector('a[href^="/@"]')) return true;
    }
    return false;
  };

  const getButtons = () =>
    [...document.querySelectorAll('button')].filter(
      (b) => LABELS.includes(b.innerText.trim()) && !clicked.has(b) && isUserRow(b)
    );

  // Walk up from a button to find the scrollable list container
  const findScroller = (el) => {
    for (let p = el.parentElement; p; p = p.parentElement) {
      if (p.scrollHeight > p.clientHeight + 20 &&
          ['auto', 'scroll'].includes(getComputedStyle(p).overflowY)) return p;
    }
    return null;
  };

  let scroller = null;
  let emptyRounds = 0;
  let count = 0;

  while (emptyRounds < 3) {
    const buttons = getButtons();

    if (buttons.length) scroller = findScroller(buttons[0]) || scroller;

    if (buttons.length === 0) {
      if (!scroller) {
        console.log('No list found. Open your Following list first.');
        break;
      }
      const before = scroller.scrollHeight;
      scroller.scrollTop = scroller.scrollHeight;
      await delay(2500);
      emptyRounds = scroller.scrollHeight === before ? emptyRounds + 1 : 0;
      continue;
    }

    emptyRounds = 0;
    const button = buttons[0];
    clicked.add(button);
    button.scrollIntoView({ block: 'center' });
    button.click();
    await delay(800);

    // Some accounts may show a confirmation
    const confirm = [...document.querySelectorAll('button')].find(
      (b) => b.innerText.trim() === 'Unfollow'
    );
    if (confirm) confirm.click();

    count++;
    console.log(`Unfollowed ${count}`);

    await delay(4000 + Math.random() * 4000); // 4-8s

    if (count % 15 === 0) await delay(90000); // 90s break every 15
  }

  console.log(`Done. Unfollowed ${count} accounts.`);
})();
```

## Tips

- To stop a script early, refresh the page.
- Both platforms limit how many accounts you can unfollow per hour and per day. Do a few dozen to ~100 per session and spread large lists over several days.
- If a script can't find the list, the platform probably changed its markup. Inspect the list in DevTools and update the selector logic.
- If you see an "Action Blocked", CAPTCHA, or "too many actions" message, stop and wait 24+ hours.

## License

This project is licensed under the [MIT License](./LICENSE). You are free to use, copy, modify, and share it, as long as the original copyright notice is included.

The software is provided "as is", without warranty of any kind. The author is not responsible for any account restrictions, action blocks, bans, or other consequences resulting from its use.

## Disclaimer

This project is for personal and educational use only. It is not affiliated with, endorsed by, or sponsored by Instagram, TikTok, Meta Platforms, Inc., or ByteDance Ltd. All trademarks belong to their respective owners. You are responsible for complying with each platform's Terms of Use.

## Contributing

Platforms change their markup often. If a script stops working, issues and pull requests with updated selectors are welcome.
