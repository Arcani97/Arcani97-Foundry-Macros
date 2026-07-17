
[![home](https://img.shields.io/badge/Home-242526?logo=googlehome)](https://github.com/Arcani97/Arcani97-Foundry-Macros)

# Mapeador de Propriedades de Itens
Um código simples para criar uma macro que verifica e gera uma lista de todas as propriedades de itens para o sistema usado no seu mundo.

Para usar, basta criar uma nova macro do tipo script e colar o código abaixo. Salve e execute a macro, e pronto: você terá uma lista de todas as propriedades dos itens do sistema instalado no mundo.

As imagens abaixo mostram a macro rodando com o sistema Symbaroum. Note que a macro só lê itens criados no mundo; antes de executá-la, crie pelo menos um item de cada tipo e preencha todos os campos disponíveis.

<br/>

<p align="center">
  <img src="https://i.imgur.com/cqmjby7.png" width="45%" alt="Collapsed List" />
  <img src="https://i.imgur.com/M2wmM1Y.png" width="45%" alt="Expanded List" />
</p>

<br/>

```JavaScript
// Função auxiliar para mapear chaves do objeto recursivamente
function getDeepKeys(obj, prefix = '') {
    let keys = [];
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            const fullPath = prefix ? `${prefix}.${key}` : key;
            keys.push(fullPath);
            
            // Se for um objeto puro (e não nulo ou array), investiga mais a fundo
            if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
                keys = keys.concat(getDeepKeys(obj[key], fullPath));
            }
        }
    }
    return keys;
}

// Execução principal
(async () => {
    const allItems = game.items;
    
    if (allItems.size === 0) {
        ui.notifications.warn("Nenhum item encontrado no diretório de itens para mapear!");
        return;
    }

    const propertiesByType = {};

    // Itera por todos os itens do mundo
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

    // Constrói o conteúdo HTML para o Pop-up
    let htmlContent = `
        <div style="font-family: monospace; max-height: 400px; overflow-y: auto; padding: 10px; background: #1a1a1a; color: #dcdcdc; border-radius: 5px;">
            <p style="margin-bottom: 15px; color: #ffb86c; font-weight: bold;">
                Mapeamento concluído. Use o botão abaixo para copiar a lista completa!
            </p>
    `;

    let rawTextToCopy = ""; // Irá armazenar o texto puro para o botão de copiar

    for (const [type, props] of Object.entries(propertiesByType)) {
        const sortedProps = Array.from(props).sort();
        
        rawTextToCopy += `=== Tipo de Item: ${type} ===\n` + sortedProps.join('\n') + `\n\n`;

        // O atributo "open" foi removido aqui para que os diretórios comecem recolhidos
        htmlContent += `
            <details style="margin-bottom: 15px; border: 1px solid #444; border-radius: 4px; padding: 5px 10px;">
                <summary style="font-weight: bold; color: #8be9fd; cursor: pointer; outline: none; user-select: none;">
                    ${type} (${sortedProps.length} propriedades)
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

    // Renderiza usando a estrutura Application V2
    new foundry.applications.api.DialogV2({
        window: {
            title: "Mapeador de Propriedades de Itens"
        },
        position: {
            width: 500,
            height: "auto"
        },
        content: htmlContent,
        buttons: [
            {
                action: "copy",
                label: "Copiar Tudo",
                icon: "fas fa-copy",
                callback: (event, button, dialog) => {
                    navigator.clipboard.writeText(rawTextToCopy.trim());
                    ui.notifications.info("Lista copiada para a área de transferência!");
                }
            },
            {
                action: "close",
                label: "Fechar",
                icon: "fas fa-times"
            }
        ]
    }).render(true);
})();
```
