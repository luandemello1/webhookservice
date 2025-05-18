## Integração com Webhooks do Discord

### Configuração do Webhook

1. No Discord:
   - Vá para as configurações do canal onde deseja receber as mensagens
   - Clique em "Integrações"
   - Clique em "Criar Webhook"
   - Copie a URL do webhook gerada

### Exemplo de Envio de Mensagem para Webhook

```lua
local HttpService = game:GetService("HttpService")

local function enviarMensagemDiscord(webhookUrl, mensagem)
    local payload = {
        content = mensagem,
        username = "Roblox Game", -- Nome que aparecerá no Discord
        avatar_url = "https://tr.rbxcdn.com/..." -- URL da imagem do avatar (opcional)
    }
    
    local sucesso, resposta = pcall(function()
        return HttpService:PostAsync(
            webhookUrl,
            HttpService:JSONEncode(payload)
        )
    end)
    
    if sucesso then
        print("Mensagem enviada com sucesso!")
    else
        warn("Erro ao enviar mensagem:", resposta)
    end
end
```

### Exemplo de Mensagem com Embed

```lua
local HttpService = game:GetService("HttpService")

local function enviarMensagemComEmbed(webhookUrl)
    local embed = {
        {
            title = "Evento no Jogo",
            description = "Um jogador completou uma missão!",
            color = 5814783, -- Cor em decimal (ex: azul)
            fields = {
                {
                    name = "Jogador",
                    value = "NomeDoJogador",
                    inline = true
                },
                {
                    name = "Pontuação",
                    value = "1000",
                    inline = true
                }
            },
            footer = {
                text = "Roblox Game - " .. os.date("%d/%m/%Y %H:%M:%S")
            }
        }
    }
    
    local payload = {
        embeds = embed
    }
    
    local sucesso, resposta = pcall(function()
        return HttpService:PostAsync(
            webhookUrl,
            HttpService:JSONEncode(payload)
        )
    end)
    
    if sucesso then
        print("Embed enviado com sucesso!")
    else
        warn("Erro ao enviar embed:", resposta)
    end
end
```

### Exemplo de Uso em Eventos do Jogo

```lua
local HttpService = game:GetService("HttpService")
local webhookUrl = "WEBHOOK_URL"

-- Exemplo de uso quando um jogador entra no jogo
game.Players.PlayerAdded:Connect(function(player)
    local payload = {
        embeds = {
            {
                title = "Novo Jogador",
                description = player.Name .. " entrou no jogo!",
                color = 65280, -- Verde
                fields = {
                    {
                        name = "ID do Jogador",
                        value = tostring(player.UserId),
                        inline = true
                    },
                    {
                        name = "Conta Criada",
                        value = player.AccountAge .. " dias atrás",
                        inline = true
                    }
                }
            }
        }
    }
    
    local sucesso, resposta = pcall(function()
        return HttpService:PostAsync(
            webhookUrl,
            HttpService:JSONEncode(payload)
        )
    end)
end)

-- Exemplo de uso quando um jogador sai do jogo
game.Players.PlayerRemoving:Connect(function(player)
    local payload = {
        embeds = {
            {
                title = "Jogador Saiu",
                description = player.Name .. " saiu do jogo!",
                color = 16711680, -- Vermelho
                timestamp = HttpService:JSONEncode(os.date("!%Y-%m-%dT%H:%M:%SZ"))
            }
        }
    }
    
    local sucesso, resposta = pcall(function()
        return HttpService:PostAsync(
            webhookUrl,
            HttpService:JSONEncode(payload)
        )
    end)
end)
```

### Dicas para Webhooks

1. **Segurança**:
   - Nunca expon a URL do webhook no código do cliente
   - Armazene a URL em um servidor seguro ou use variáveis de ambiente
   - Considere usar um servidor intermediário para validar as requisições

2. **Rate Limiting**:
   - O Discord tem limites de requisições por minuto
   - Implemente um sistema de fila para evitar exceder os limites
   - Considere agrupar mensagens para reduzir o número de requisições

3. **Formatação**:
   - Use embeds para mensagens mais complexas
   - Mantenha as mensagens concisas e informativas
   - Use cores diferentes para diferentes tipos de eventos

4. **Tratamento de Erros**:
```lua
local function enviarMensagemSegura(webhookUrl, payload)
    local tentativas = 0
    local maxTentativas = 3
    
    while tentativas < maxTentativas do
        local sucesso, resposta = pcall(function()
            return HttpService:PostAsync(
                webhookUrl,
                HttpService:JSONEncode(payload)
            )
        end)
        
        if sucesso then
            return true
        end
        
        tentativas = tentativas + 1
        wait(1) -- Espera 1 segundo antes de tentar novamente
    end
    
    warn("Falha ao enviar mensagem após", maxTentativas, "tentativas")
    return false
end
```

### Exemplo de Sistema de Logs Completo

```lua
local HttpService = game:GetService("HttpService")
local webhookUrl = "WEBHOOK_URL"

local LogsService = {
    webhookUrl = webhookUrl,
    
    cores = {
        sucesso = 65280,    -- Verde
        erro = 16711680,    -- Vermelho
        info = 5814783,     -- Azul
        aviso = 16776960    -- Amarelo
    },
    
    enviarLog = function(self, titulo, descricao, tipo, campos)
        local payload = {
            embeds = {
                {
                    title = titulo,
                    description = descricao,
                    color = self.cores[tipo] or self.cores.info,
                    fields = campos or {},
                    timestamp = HttpService:JSONEncode(os.date("!%Y-%m-%dT%H:%M:%SZ"))
                }
            }
        }
        
        return enviarMensagemSegura(self.webhookUrl, payload)
    end
}

-- Exemplo de uso
LogsService:enviarLog(
    "Missão Completada",
    "Um jogador completou uma missão difícil!",
    "sucesso",
    {
        {
            name = "Jogador",
            value = "NomeDoJogador",
            inline = true
        },
        {
            name = "Missão",
            value = "Missão Secreta",
            inline = true
        },
        {
            name = "Recompensa",
            value = "1000 moedas",
            inline = true
        }
    }
)
```
