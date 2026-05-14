<vbox flex />
<vbox class="settings">
  <HeaderGroupBox>
    <hbox slot="header">{$t`Import / Export`}</hbox>
    <hbox class="buttons">
      <Button
        label={$t`Import from iCal .ics file…`}
        onClick={importICal}
        />
      <Button
        label={$t`Export to iCal .ics file…`}
        onClick={exportICal}
        />
    </hbox>
  </HeaderGroupBox>
</vbox>

<FileSelector bind:this={importFileSelector} acceptFileTypes={["text/calendar"]} />

<script lang="ts">
  import { Calendar } from "../../../logic/Calendar/Calendar";
  import { convertICalToEvents } from "../../../logic/Calendar/ICal/ICalToEvent";
  import { eventsToICalFile } from "../../../logic/Calendar/ICal/ICalGenerator";
  import FileSelector from "../../Mail/Composer/Attachments/FileSelector.svelte";
  import HeaderGroupBox from "../../Shared/HeaderGroupBox.svelte";
  import Button from "../../Shared/Button.svelte";
  import { saveBlobAsFile } from "../../Util/util";
  import { logError } from "../../Util/error";
  import { t } from "../../../l10n/l10n";
  import { ArrayColl } from "svelte-collections";

  export let account: Calendar;

  let importFileSelector: FileSelector;
  async function importICal() {
    let file = await importFileSelector.selectFile();
    if (!file) {
      return;
    }
    let fileContents = await file.text();
    let errors = new ArrayColl<Error>();
    let events = convertICalToEvents(fileContents, () => account.newEvent(), errors);
    account.events.addAll(events);
    for (let event of events) {
      try {
        await event.save();
      } catch (ex) {
        errors.add(ex);
      }
    }
    await account.save();

    if (errors.hasItems) {
      for (let ex of errors) {
        logError(ex);
      }
      throw errors.first;
    }
  }

  async function exportICal() {
    let file = eventsToICalFile(account.events, account.name);
    await saveBlobAsFile(file);
  }
</script>

<style>
  .settings {
    align-items: start;
    justify-content: start;
  }
  .settings > :global(.group) {
    width: 100%;
  }
  .buttons {
    align-items: center;
    gap: 12px;
  }
</style>
