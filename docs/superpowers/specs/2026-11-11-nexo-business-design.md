# NEXO Business — desenho da primeira versão

Data: 11/11/2026. Estado: proposta para aprovação; não é implementação.

## Intenção
Um produto real para pequenos empresários organizarem o atendimento. A empresa escolhe seu segmento no cadastro e vê linguagem e módulos adequados. A primeira entrega atende prestadores de serviço com agendamentos; outros segmentos entram depois, sem prometer atendimento universal nesta versão.

## Objetivo e critério de sucesso
Um proprietário consegue criar sua empresa, acessar com segurança, cadastrar serviços e clientes, criar/editar/cancelar agendamentos e consultar a agenda no celular. Outra empresa não pode ler nem alterar esses registros. A AUREON pode demonstrar o produto com dados fictícios sem expor clientes reais.

## Abordagens consideradas
1. Base comum com módulos por segmento — escolhida. Compartilha autenticação, empresas, clientes e configurações; habilita agenda para prestadores de serviço, e módulos futuros para lojas. Lançamento incremental e manutenção centralizada.
2. Aplicativos separados por segmento — descartada por duplicação de código, manutenção e operação, embora permita personalização profunda.

## Primeira versão
- PWA responsivo, instalável, com marca NEXO Business e crédito discreto “por AUREON SaaS”.
- Cadastro/login por e-mail, recuperação de acesso e sessão persistente; nenhum acesso a dados sem autenticação.
- Criação de uma empresa e escolha do segmento “Serviços com agendamento”. O proprietário é o administrador da empresa.
- Painel: agendamentos do dia, próximos horários e atalhos para novo cliente/serviço/agendamento.
- Clientes: nome, contato opcional, observação; busca e edição.
- Serviços: nome, duração, preço opcional e ativo/inativo.
- Agenda: data/hora, cliente, serviço, status (agendado, concluído, cancelado), observações; edição e prevenção de conflito de horário no mesmo recurso/profissional, se este for modelado.
- Configurações: dados básicos da empresa e fuso horário. Idioma português do Brasil e moeda BRL.
- Navegação móvel clara e estados vazios úteis; formulário com validação e mensagens de erro.

## Dados e segurança
Entidades mínimas: usuário, empresa, vínculo usuário-empresa com papel, cliente, serviço e agendamento. Toda leitura/escrita de dados operacionais deve ser restrita pelo identificador da empresa no servidor/banco, nunca apenas por filtro visual. Senhas não são armazenadas no app; credenciais e chaves privadas não entram no GitHub. Chave pública de cliente, se necessária, não substitui políticas de acesso. Backup, migrações, logs sem dados sensíveis e teste de acesso cruzado são pré-requisitos de produção. Política de privacidade e exclusão/exportação de dados devem ser definidas antes da comercialização.

## Limites desta entrega
Sem pagamentos, WhatsApp automático, marketplace, IA geradora de aplicativos, painel administrativo global, múltiplas unidades, estoque, pedidos de loja ou calendário de terceiros. Não anunciar estes recursos como disponíveis. Não migrar automaticamente dados de outros aplicativos AUREON.

## Implementação proposta
Frontend web/PWA no repositório NEXO-BUSINESS; autenticação e banco gerenciados em serviço separado com políticas por empresa. A escolha do provedor exige auditoria de maturidade do “Aureon Base” antes de conectá-lo; não se deve substituir segurança pronta por um backend ainda não comprovado. Ambiente de teste separado da produção, variáveis de ambiente no host, CI com testes e publicação somente após validação. GitHub Pages serve frontend estático, mas não hospeda o backend.

## Fluxos de aceite
1. Usuário cria conta e empresa de serviços; entra e sai; recupera acesso.
2. Cria um cliente e serviço, agenda e vê no painel; edita e cancela, com dados persistidos após recarregar.
3. Dois usuários de empresas diferentes não conseguem consultar ou alterar dados alheios, inclusive por chamada direta à API.
4. Uso em largura de celular e instalação PWA são testados.
5. Nenhuma chave privada, senha, dado real de cliente ou regra de acesso apenas no navegador é publicada.
6. Links, termos e privacidade, mensagens de erro e backup/restauração são verificados antes de liberar clientes pagantes.

## Decisões ainda necessárias
- Provedor de autenticação e banco: auditar Aureon Base e, se não cumprir isolamento, recuperação e operação, usar um serviço gerenciado comprovado para a primeira versão.
- Se agenda inicial terá um único profissional/recurso por empresa ou vários; default proposto: um recurso, para limitar escopo.
- Domínio e hospedagem de produção; não afirmar que o link está no ar antes de configurar e testar.
