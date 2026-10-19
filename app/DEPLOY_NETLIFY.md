# Publicar no Netlify com banco de dados permanente (MongoDB Atlas)

## Como funciona
- **Site (telas)**: publicado no Netlify (pasta `frontend/build`).
- **Servidor**: Netlify Functions (`frontend/netlify/functions/api.mjs`), que respondem em `/api/*`.
- **Banco de dados**: **MongoDB Atlas** (plano gratuito M0, 512 MB), fora do Netlify.
- O Netlify **não guarda nenhum dado**. Cada nova publicação troca só as telas e o código; os cadastros continuam no Atlas.
- Todos os aparelhos acessam o mesmo banco, então veem os mesmos dados. Não é usado o armazenamento local do navegador.

## 1. Criar o banco no MongoDB Atlas (grátis)
1. Crie uma conta em https://www.mongodb.com/cloud/atlas/register
2. **Create** → escolha **M0 (Free)** → região **São Paulo (sa-east-1)** → Create.
3. **Database Access** → **Add New Database User**:
   - Usuário: `festa_app`; senha forte (gere uma e guarde).
   - Privilégio: **Specific Privileges** → `readWrite` no banco `festa_padroeira` (só esse banco, nada de admin).
4. **Network Access** → **Add IP Address** → `0.0.0.0/0`. O Netlify não tem IP fixo, então é preciso liberar todos. O acesso continua protegido pelo usuário e pela senha do banco.
5. **Database** → **Connect** → **Drivers**. Copie o endereço, que tem este formato:
   `mongodb+srv://festa_app:SUA_SENHA@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority`

## 2. Publicar no Netlify
1. Use **Save to GitHub** para mandar o código para um repositório.
2. No Netlify: **Add new site → Import from Git** → escolha o repositório.
   O arquivo `netlify.toml` (na raiz) já configura tudo: pasta base, comando de build e funções.
3. **Site configuration → Environment variables** → adicione (marque como *secret*):
   | Variável | Valor |
   |---|---|
   | `MONGO_URL` | o endereço copiado do Atlas |
   | `DB_NAME` | `festa_padroeira` |
   | `ADMIN_PASSWORD` | a senha de gestão de setores / backup |
4. **Deploy**. No primeiro acesso, os setores e capelas são criados automaticamente no banco.

## Segurança configurada
- Endereço e senha do banco, e senha de administrador, ficam **só nas variáveis do Netlify**, nunca no código.
- O usuário do banco só pode ler e gravar no banco do app.
- Gestão de setores e backup exigem a senha de administrador. Depois de 5 senhas erradas, aquele IP fica bloqueado por 15 minutos.
- Os campos têm limite de tamanho, a busca é protegida contra expressões maliciosas e IDs inválidos retornam 404.
- Cabeçalhos de segurança (HSTS, anti-clickjacking, nosniff) estão definidos no `netlify.toml`, e a conexão é sempre HTTPS.
- Cadastro e consulta continuam abertos a todos, como hoje.

## Como os dados ficam preservados
- Ficam no Atlas, que é independente do Netlify. Republicar, atualizar ou até apagar o site não apaga os dados.
- Botão **"Baixar backup"** (aba Cadastros, com senha): baixa um arquivo `.json` com todos os setores e cadastros. Faça isso com frequência.
- A **Planilha** (Excel) continua disponível para relatórios.
- Atenção: o plano M0 grátis **não** tem backup automático do Atlas. Por isso o botão de backup é importante.
