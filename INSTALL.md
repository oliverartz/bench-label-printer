# Install and use Bench Label Printer 0.4.2

You need a Mac running macOS 14 or later and a network connection to a Brady BBP33. The printer needs B33-152-492 labels and a B30-R4300 or B30-R6400 ribbon. Ask your lab for the printer address.

## Install on the Mac

1. Check **Apple menu → About This Mac**. Choose the **Apple silicon** disk image for an Apple M-series Mac or **Intel** for an Intel processor. [v0.4.2 downloads and SHA-256 checksums](https://github.com/oliverartz/bench-label-printer/releases/tag/v0.4.2)
2. Open the disk image and drag **Bench Label Printer.app** into **Applications**. Eject the disk image and open the installed app. If upgrading from Brady Labels, quit the old app first. The new app reads your saved settings and jobs.
3. First launch may need your approval because the app is ad-hoc signed and Apple has not notarized it. Follow Apple's first-launch instructions if your Mac allows it. On a managed Mac, contact IT if the option is unavailable. [Apple first-launch instructions](https://support.apple.com/en-us/102445)
4. In **Settings**, save your initials and the BBP33 address supplied by your lab, then enable printing. Allow local network access if macOS asks. Choose **Check materials**. If the check fails, verify the printer address, network, stock, and ribbon before continuing.

For a manual test, add `TEST LEFT` and `TEST RIGHT` as two designs, each with **Copies** set to 1. Choose **Prepare labels**, review **2 labels · 1 row**, then print once. Check the physical row for clear, correctly aligned labels in both columns.

The **Third-Party Licenses** folder is in the disk image and in the installed app bundle under `Contents/Resources`.

## Use with a local AI assistant

Paste the link to this guide and the following prompt into an assistant that can use Terminal on the destination Mac. A browser-only chat can walk you through the Finder and app steps.

> Help me install Bench Label Printer v0.4.2 using this guide. Check my macOS version and Mac chip, use the matching official release and verify its checksum, then help me enter my printer details in Settings. Prepare the two-label test and show me the preview before sending anything to the printer. Ask me to handle macOS permission prompts.

An assistant can use `sw_vers -productVersion` to check macOS and `sysctl -n hw.optional.arm64` to identify Apple silicon (value 1, including when Terminal is running under Rosetta). Confirm the chip in **About This Mac** if needed. To verify a download, use `shasum -a 256 FILE.dmg` and compare the exact filename and hash against `SHA256SUMS.txt` from the release. Use Finder for the install or attach the image read-only with `hdiutil`, copy the app into **Applications** with `ditto`, and detach the image. If an app with the same name is already installed, ask before replacing it.

The app includes a command-line tool. These commands show its options and supported stock without contacting the printer. Adjust `APP` if you installed it elsewhere.

```sh
APP="/Applications/Bench Label Printer.app"
CLI="$APP/Contents/Resources/brady-label"
"$CLI" --help
"$CLI" profile
```

After the user enters the address in **Settings**, `"$CLI" materials` checks the actual stock and ribbon without printing. Prepare the two-label test offline with:

```sh
TEST_DIR=$(mktemp -d)
cat > "$TEST_DIR/labels.json" <<'JSON'
{"labels":[{"text":"TEST LEFT","copies":1},{"text":"TEST RIGHT","copies":1}]}
JSON
"$CLI" batch "$TEST_DIR/labels.json" --preview work/test-row.svg
```

Show the SVG at the returned `preview_file` path. Confirm the response has `label_count: 2`, `rows: 1`, and `print_data_sent: false`. Save the returned `id`. `"$CLI" inspect JOB_ID` displays the prepared job. Replace `JOB_ID` with that ID.

Submit once with `"$CLI" send JOB_ID --send` only after the person at the Mac asks to print that job and the materials check passes. If the result says `submitted_unconfirmed`, check the physical labels. If a submission fails or the result is unclear, inspect the printer before attempting another job.
