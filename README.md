# EDITORIAL BRAND — base de loja editorial digital

Starter de frontend React + TypeScript + Vite, com conteúdo em português, moeda em Kz, landing page de produto, checkout, páginas institucionais e base de backend Supabase.

## Estado real desta entrega

**Implementado no projeto:** interface responsiva; página inicial; página individual do ebook; estrutura reutilizável de dados de produto; resumo de encomenda; formulário de checkout; páginas institucionais; página de consulta de estado; schema SQL para produtos/encomendas/eventos/entregas/auditoria; funções Supabase de base.

**Ainda não está pronto para aceitar pagamentos reais:** não foi escolhida/validada uma API de gateway; não existem credenciais comerciais; o webhook rejeita eventos enquanto não for implementada a validação oficial; a entrega/download protegido e o email transacional precisam de ser concluídos; não há painel de administração completo nem analítica ligada. Não publicar como loja operacional até concluir e testar estes pontos. Nenhum pagamento é simulado ou marcado como pago.

## Executar localmente

1. Instala Node.js LTS.
2. Descompacta o projeto e executa `npm install`.
3. Copia `.env.example` para `.env.local`.
4. Para ver a interface, executa `npm run dev`.
5. Para verificar a compilação, executa `npm run build`.

Sem variáveis Supabase, o checkout explica que ainda não está ligado e não inicia pagamentos.

## Configuração Supabase

1. Cria um projeto Supabase.
2. Executa `supabase/schema.sql` no SQL Editor.
3. Mantém o bucket `digital-products` privado. Carrega o PDF real por um fluxo administrativo seguro e guarda o caminho em `products.file_path`. Não publicar os ficheiros originais.
4. Define `VITE_SUPABASE_URL` e `VITE_SUPABASE_ANON_KEY` localmente. A chave anon pode estar no frontend se as políticas RLS estiverem corretas; nunca exponhas `service_role`.
5. Instala Supabase CLI e liga o projeto.
6. Define `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `PUBLIC_SITE_ORIGIN` e os segredos do fornecedor como secrets das Edge Functions, nunca em `VITE_*`.
7. Faz deploy das funções `create-order`, `payment-webhook`, `order-status` e `digital-download`.

O schema cria o produto de exemplo como **não publicado** (`is_published=false`) e a capa/ficha no frontend é conteúdo editorial de demonstração. Revê e substitui tudo pelo produto real antes de publicar. Para publicar um produto após validar conteúdo e ficheiro, usa o painel administrativo seguro que deverá ser implementado, ou uma operação de servidor controlada — não abras escrita pública na tabela.

## Integração de pagamentos — bloqueio obrigatório antes de produção

Escolhe um fornecedor que aceite comerciantes em Angola e confirme suporte ao método desejado (por exemplo, Multicaixa Express ou referência, se disponível para a conta). Verifica diretamente a documentação oficial, onboarding, liquidação em AOA, taxas, política de webhooks, ambiente sandbox e condições comerciais. A menção a um fornecedor neste briefing não confirma que a API ou o produto estão disponíveis.

Na função `create-order`:
- obter o preço do produto no banco de dados (já preparado);
- criar encomenda com chave de idempotência;
- iniciar checkout usando o endpoint documentado oficialmente;
- guardar referência do fornecedor e devolver apenas a URL de checkout validada.

Na função `payment-webhook`:
- validar assinatura sobre o corpo bruto conforme documentação oficial;
- validar o evento e confirmar a transação server-to-server;
- conferir referência, montante, moeda, estado e encomenda;
- gravar o ID do evento com restrição única;
- processar eventos repetidos de forma idempotente;
- só marcar `paid` depois de validação completa;
- nunca aceitar o regresso à página de sucesso como prova de pagamento.

Não inventar endpoints nem considerar um webhook válido só porque contém um campo `success`.

## Entrega digital e email

- Criar fluxo de recuperação que verifique controlo do email (OTP ou token de uso único com expiração).
- Confirmar que a encomenda está `paid` antes de emitir link assinado de curta duração para o bucket privado.
- Registar a entrega na tabela `deliveries`; aplicar idempotência.
- Configurar fornecedor de email transacional, templates e política de retry.
- Se email não for válido/disponível, oferecer recuperação segura através de suporte, sem tornar o ficheiro público.

## Administração, conteúdo e operação

Antes do lançamento, implementar um painel protegido por autenticação e autorização de administrador para CRUD de produtos, preço, ficheiro privado, conteúdo da landing page, estado de publicação e histórico de encomendas. Não confiar apenas em esconder rotas no frontend; aplicar autorização no backend. Os textos legais atuais são modelos iniciais e requerem revisão jurídica.

## Verificação antes de produção

- [ ] Rever identidade/nome final e conteúdo real do ebook.
- [ ] Carregar PDF real e confirmar que o bucket é privado.
- [ ] Rever RLS e testar com sessão anónima.
- [ ] Implementar e validar gateway segundo documentação oficial.
- [ ] Testar webhook com assinatura inválida, eventos repetidos e montantes divergentes.
- [ ] Testar pagamentos pendentes, falhados, expirados e confirmados.
- [ ] Confirmar que o redirect não liberta acesso sem webhook validado.
- [ ] Testar recuperação e links de download expirados.
- [ ] Configurar email transacional e suporte real.
- [ ] Concluir painel de administração, logs, backups, monitorização e analítica com consentimento apropriado.
- [ ] Rever privacidade, termos e reembolsos para Angola e mercados futuros.
