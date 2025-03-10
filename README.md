# Instruções

Você foi contratado por uma empresa de fast-food para resolver um grande problema usando inteligência artifical.

Devido à falta de funcionários, a empresa está com dificuldades para atender a todos os pedidos. Para resolver esse problema, a empresa decidiu criar um robô capaz de tirar dúvidas a respeito dos itens do cardápio e anotar pedidos de clientes.

## Arquitetura do sistema

Este problema chegou no seu Tech Lead. Ele identificou que o que estava sendo pedido era um agente que usa IA generativa. Ele desenhou a arquitetura do sistema em 3 aplicações que consistem em sistemas para interagir com um AI agent.

O líder técnico conversou com outros setores de desenvolvimento e lhe passou a missão de resolver esse problema com a seguinte estrutura:

### 1. Frontend

Você vai precisar desenvolver um frontend em React (sob Next.js) capaz de fazer requisições para as APIs. A ideia é construir uma interface com uma entrada de texto e histórico de mensagens (utilize paginação). O seu frontend precisará, portanto, de 3 telas: uma de criar conta, uma de login e uma de chat.

### 2. Core API

Buscando otimizar a performance da aplicação, seu líder decidiu que a aplicação que cuidará de autenticação e de CRUDs no banco de dados será uma aplicação Golang, usando Fiber.

O time chegou a conclusão de que seria necessário a implementação de documentação usando swagger, então você usará o swaggo para gerar documentação e sua API deverá implementar corretamente as ferramentas de documentação.

Alguns requisitos foram definidos para a API:

1. Autenticação usando OAuth2;
2. Você precisará usar GORM para fazer as operações no banco de dados;
3. Você usará Postgres como banco de dados.

Como o líder técnico é macaco velho e já realizou a tarefa de construir um backend com esse propósito incontáveis vezes, ele já havia montado o repositório [core-api-template](https://github.com/cogniia/core-api-template) onde você encontrará um template para a API que lhe ajudará muito. Você deverá fazer um fork desse repositório e implementar a API.

### 3. Agent API

Visando a versatilidade da linguagem Python dentro da IA generativa, o time de AI enginnering decidiu que a aplicação que cuidará de gerar respostas para os clientes será uma aplicação Python, usando FastAPI.

Como o alto escalão está muito ocupado em reuniões e rolê gastronômico, você ficou responsável por implementar a API. Alguns requisitos foram definidos:

1. Você deverá usar FastAPI e se aproveitar da documentação automática;
2. Sua aplicação FastAPI será responsável por rodar um grafo construído com Langgraph;
3. Desse modo, você vai construir um agent capaz de
   a. Conversar com o usuário sobre itens do cardápio, tirando dúvidas, etc;
   b. Anotar pedidos que devem ser enviados para a Core API e salvos no Banco de Dados.
4. As informações sobre os pedidos são todas tiradas de um banco de dados vetorial, que contém informações sobre os itens do cardápio;
5. A autenticação é feita com middlewares que mandam requests para a Core API para garantir que o usuário está autenticado. Você vai querer usar algo como

   ```python
   def user_middleware(
    credentials: HTTPAuthorizationCredentials = Depends(HTTPBearer()),
   ) -> Tuple[User, str]:
    """
    Dependency that retrieves the current user using the provided bearer token.
    """
    # Construct the full authorization header value (e.g., "Bearer <token>")
    token = f"{credentials.scheme} {credentials.credentials}"
    print(f"authorization token is {token}")
    try:
        # make the request here
        return user, token
    except Exception as exc:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
        ) from exc
   ```

   Portanto, você usará o OAuth2 implementado na CoreAPI;

6. Obviamente, seu agent deve usar algum LLM para gerar respostas. Escolha o que achar melhor. Você ganha pontos se sua aplicação for agnóstica de modelo.

## Observações

1. Temos que ser capazes de rodar o projeto localmente. Portanto, você deve fornecer instruções claras de como rodar o projeto e também variáveis de ambiente para fazer isso. Procure usar Docker para coisas como banco de dados, RabbitMQ, etc.
2. Você deve fornecer instruções e dados para o banco de dados vetorial para que possamos testar localmente.
3. Vamos considerar a organização do seu código, comentários, a qualidade da sua documentação, etc.
4. Você deve seguir os padrões de um projeto que será realmente usado em produção. Obviamente que a qualidade não será a mesma, mas estamos falando de boas práticas como usar variáveis de ambiente e esconder o `.env` com `.gitignore`.
5. Para salvar os pedidos, você provavelmente vai ter que criar um endpoint na Core API e uma habilidade do agent de salvar o pedido usando esse API. Pense em como você vai fazer isso.
6. Provavelmente o seu agente vai ter que tomar uma decisão se deve salvar o pedido ou apenas responder a mensagem com o RAG. Você terá que usar edges condicionais no Langgraph para isso. Novamente, pense em como você vai fazer isso.
7. Vamos executar o seu projeto, criar uma conta, testar o agent com a UI que você criou e em seguida avaliar o seu código. Se não conseguirmos rodar o projeto localmente, nem leremos o código.
