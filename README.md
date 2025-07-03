# artigos-desenvolvimento

💡 Visão Geral
 

Este documento descreve a transição do método de exclusão de usuários de hard delete (remoção física) para soft delete (marcação lógica). A mudança foi motivada pela necessidade de corrigir erros 404 em documentações associadas a usuários deletados, garantindo a integridade dos dados e preservando o conhecimento organizacional.

 

🚨 O Problema: Conteúdo Perdido e Links Quebrados
 

Anteriormente, ao excluir um usuário, o registro era permanentemente removido do banco de dados. Essa abordagem, conhecida como hard delete, gerava efeitos colaterais críticos:

Links Quebrados: Documentações criadas pelo usuário excluído permaneciam no sistema, mas as referências de autoria (created_by, updated_by) apontavam para um ID inexistente. Isso resultava em erros 404 ao tentar acessar ou carregar metadados dessas páginas.

Perda de Integridade: A exclusão física quebrava a integridade referencial do banco de dados, dificultando a rastreabilidade e auditoria.

Experiência do Usuário Negativa: Os usuários encontravam páginas de erro inesperadamente, gerando frustração e desconfiança na plataforma.

Perda de Conhecimento: O conteúdo criado por ex-colaboradores corria o risco de se tornar inacessível, representando uma perda de conhecimento valioso para a empresa.

 

 

✅ A Solução: Implementando o Soft Delete
 

Para resolver esses problemas, adotamos o padrão Soft Delete. Em vez de apagar o registro, nós simplesmente alteramos o status do usuário para DELETED. O usuário se torna inativo e invisível nas interfaces do sistema, mas seu registro é preservado para manter a integridade das referências.

 

Alterações Técnicas Realizadas
 

A implementação foi focada em três áreas principais do código:

 

1. Atualização do Schema do Banco de Dados
 

Adicionamos o novo status DELETED ao enum de status de usuário, permitindo que o banco de dados acomode essa nova condição.

📍 Arquivo: server/db/schema.ts

 

Ver código ANTES e DEPOIS

 



TypeScript


// ANTES
status: text('status', {
  enum: ['ACTIVE', 'PENDING', 'BLOCKED', 'INACTIVITY'],
}).$default(() => 'PENDING'),
// DEPOIS
status: text('status', {
  enum: ['ACTIVE', 'PENDING', 'BLOCKED', 'INACTIVITY', 'DELETED'],
}).$default(() => 'PENDING'),
 

 

 

2. Modificação da Função de Exclusão
 

A lógica de negócio para deletar um usuário foi alterada. Em vez de executar um db.delete(), agora executamos um db.update() para atribuir o status DELETED.

📍 Arquivo: actions/user.ts

 

Ver código ANTES e DEPOIS

 



TypeScript


// ANTES - Exclusão física (Hard Delete)
export const DeleteUserById = async (id: string) => {
  // ... validações ...
  try {
    await db.delete(users).where(eq(users.id, id))
    return { success: 'Usuário deletado com sucesso' }
  } catch {
    return { error: 'Erro ao deletar usuário' }
  }
}
// DEPOIS - Marcação lógica (Soft Delete)
export const DeleteUserById = async (id: string) => {
  // ... validações ...
  try {
    // A linha abaixo foi substituída pela atualização de status
    // await db.delete(users).where(eq(users.id, id))
    await db.update(users).set({ status: 'DELETED' }).where(eq(users.id, id));
    return { success: 'Usuário deletado com sucesso' }
  } catch {
    return { error: 'Erro ao deletar usuário' }
  }
}
 

 

 

3. Filtro de Usuários Deletados na Listagem
 

Para garantir que os usuários "deletados" não apareçam em listagens, pesquisas ou menus de seleção, adicionamos um filtro em todas as consultas relevantes para excluir registros com status DELETED.

📍 Arquivo: lib/user.ts

 

Ver código ANTES e DEPOIS

 



TypeScript


// Adicionado import para a condição "not equal"
import { eq, ne } from 'drizzle-orm'
// ANTES - Retornava todos os usuários, independente do status
export const getAllUsersWithPartialInfo = async () => {
  try {
    const users = (await db.query.users.findMany({
      // ... colunas
    })) as ParcialUser[]
    return users
  } catch {
    return []
  }
}
// DEPOIS - Filtra para não incluir usuários deletados
export const getAllUsersWithPartialInfo = async () => {
  try {
    const usersResponse = (await db.query.users.findMany({
      where: ne(users.status, 'DELETED'), // <-- FILTRO ADICIONADO
      columns: {
        // ... colunas
      },
    })) as ParcialUser[]
    return usersResponse;
  } catch {
    return []
  }
}
 

 

 

 

🎯 Benefícios Técnicos e de Negócio
 

A adoção do Soft Delete trouxe vantagens imediatas e estratégicas:

🔒 Preservação do Conhecimento: Todo o conteúdo criado por usuários permanece acessível e com a autoria correta, protegendo o capital intelectual da empresa.

🔗 Integridade Referencial: Garante que todas as chaves estrangeiras (created_by, updated_by) permaneçam válidas, fortalecendo a consistência e a confiabilidade do banco de dados.

✨ Melhoria na Experiência do Usuário: Elimina por completo os erros 404, fornecendo uma navegação fluida e confiável.

🔧 Flexibilidade e Escalabilidade: Abre portas para funcionalidades futuras, como a reativação de usuários, transferência de propriedade de documentos e auditorias mais completas.

 

 

🚀 Guia de Uso e Recomendações
 

 

Para Administradores

 

A experiência na interface de administração permanece a mesma. O botão "Excluir Usuário" agora executa o soft delete. O usuário removido desaparecerá das listas ativas, mas suas contribuições (documentos, comentários, etc.) serão mantidas.

 

Para Desenvolvedores

 

⚠️ Atenção: Ao realizar consultas personalizadas diretamente no banco de dados que envolvam a tabela de usuários, lembre-se sempre de adicionar a condição where: ne(users.status, 'DELETED') para evitar que usuários deletados apareçam em locais inesperados. Utilize preferencialmente as funções de serviço já existentes, como getAllUsersWithPartialInfo(), que contêm este tratamento.

 

 

📊 Monitoramento e Próximos Passos
 

Para garantir a saúde contínua do sistema, recomendamos:

Monitoramento:

Criar dashboards para acompanhar o número de usuários com status DELETED.

Implementar scripts de verificação periódica para garantir a integridade de dados e identificar possíveis conteúdos órfãos.

Adicionar logs específicos para cada operação de soft delete.

Próximos Passos:

Funcionalidade de Restauração: Desenvolver uma interface ou script que permita reativar um usuário deletado.

Limpeza de Dados (LGPD): Planejar um processo para anonimizar ou remover dados sensíveis de usuários deletados após um período determinado, em conformidade com políticas de privacidade.

Dashboard de Auditoria: Criar uma visão para administradores que exiba os usuários deletados e suas documentações associadas.
