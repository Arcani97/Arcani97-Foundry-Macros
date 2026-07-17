# Item-Property-Mapper

A simple piece of code to create a macro that checks and generates a list of all item properties for the system used in the world.

To use it, simply create a new script-type macro and paste the code below. Save and run the macro, and that's it: you have a list of all the properties of the system items installed in the world.

The images below show the macro running using the Symbaroum system. Please note that the macro only reads items created in the world; before running the macro, create at least one of each item type and fill in all available fields.

<br/>

<p align="center">
  <img src="https://i.imgur.com/KEMY8wT.png" width="45%" alt="Collapsed List" />
  <img src="https://i.imgur.com/jeSatls.png" width="45%" alt="Expanded List" />
</p>

<br/>

```JavaScript
// Helper function to recursively map object keys
function getDeepKeys(obj, prefix = '') {
    let keys = [];
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            const fullPath = prefix ? `${prefix}.${key}` : key;
            keys.push(fullPath);
            
            // If it is a pure object (and not null or array), dig deeper
            if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
                keys = keys.concat(getDeepKeys(obj[key], fullPath));
            }
        }
    }
    return keys;
}

// Main execution
(async () => {
    const allItems = game.items;
    
    if (allItems.size === 0) {
        ui.notifications.warn("No items found in the items directory to map!");
        return;
    }

    const propertiesByType = {};

    // Iterate through all items in the world
    for (let item of allItems) {
        const itemType = item.type;
        const systemData = item.system;

        if (!systemData) continue;

        if (!propertiesByType[itemType]) {
            propertiesByType[itemType] = new Set();
        }

        const keys = getDeepKeys(systemData);
        keys.forEach(key => propertiesByType[itemType].add(`item.system.${key}`));
    }

    // Build HTML content for the Pop-up
    let htmlContent = `
        <div style="font-family: monospace; max-height: 400px; overflow-y: auto; padding: 10px; background: #1a1a1a; color: #dcdcdc; border-radius: 5px;">
            <p style="margin-bottom: 15px; color: #ffb86c; font-weight: bold;">
                Mapping complete. Use the button below to copy the full list!
            </p>
    `;

    let rawTextToCopy = ""; // Will hold the plain text for the copy button

    for (const [type, props] of Object.entries(propertiesByType)) {
        const sortedProps = Array.from(props).sort();
        
        rawTextToCopy += `=== Item Type: ${type} ===\n` + sortedProps.join('\n') + `\n\n`;

        // The "open" attribute was removed here so directories start collapsed
        htmlContent += `
            <details style="margin-bottom: 15px; border: 1px solid #444; border-radius: 4px; padding: 5px 10px;">
                <summary style="font-weight: bold; color: #8be9fd; cursor: pointer; outline: none; user-select: none;">
                    ${type} (${sortedProps.length} properties)
                </summary>
                <ul style="list-style-type: none; padding-left: 15px; margin: 8px 0 0 0; line-height: 1.4;">
        `;

        sortedProps.forEach(prop => {
            htmlContent += `<li style="border-bottom: 1px solid #222; padding: 2px 0;">${prop}</li>`;
        });

        htmlContent += `
                </ul>
            </details>
        `;
    }

    htmlContent += `</div>`;

    // Render using the V2 Application framework
    new foundry.applications.api.DialogV2({
        window: {
            title: "Item Property Mapper"
        },
        position: {
            width: 500,
            height: "auto"
        },
        content: htmlContent,
        buttons: [
            {
                action: "copy",
                label: "Copy All",
                icon: "fas fa-copy",
                callback: (event, button, dialog) => {
                    navigator.clipboard.writeText(rawTextToCopy.trim());
                    ui.notifications.info("List copied to clipboard!");
                }
            },
            {
                action: "close",
                label: "Close",
                icon: "fas fa-times"
            }
        ]
    }).render(true);
})();
```
