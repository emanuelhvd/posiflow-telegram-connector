Contexto:
Trabalho na Positivo S+ desenvolvendo o chatbot "Estella" usando TileDesk integrado com Telegram. Atualmente, quando usuários clicam em botões, as mensagens NÃO aparecem no histórico do chat porque o TileDesk usa Inline Keyboard (callback invisível).

Vale lembrar que essa é uma hipotese, eu imagino que o comportamento seja esteja dessa forma por causa do inline keyboard, mas posso estar errado....
Preciso que vc avalie o projeto que está na raiz desse diretorio, para me confirmar.

Objetivo:
Modificar o pacote @tiledesk/tiledesk-telegram-connector para usar Reply Keyboard ao invés de Inline Keyboard, fazendo com que cliques em botões apareçam como mensagens visíveis no histórico.

Status atual:

Já fiz fork do repositório tiledesk-telegram-connector
Já clonei localmente
Gestor autorizou a modificação do código fonte

Tarefa:
Modificar o método toTelegram() no arquivo do translator para:

Remover lógica de inline_keyboard
Implementar keyboard (Reply Keyboard)
Processar apenas botões tipo text (não URL nem Action)
Configurar one_time_keyboard: true para remover teclado após clicar

Detalhes técnicos:

Arquivo: provavelmente lib/translator.js ou similar
Classe: TiledeskTelegramTranslator
Trocar reply_markup.inline_keyboard por reply_markup.keyboard
Estrutura: keyboard: [[{text: "valor"}]] ao invés de inline_keyboard: [[{text: "x", callback_data: "y"}]]