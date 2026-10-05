# só país — tarifa base
login__cr.br:senha@gw.dataimpulse.com:823

# cidade específica — tráfego cobrado em dobro
login__cr.br;city.salvador:senha@gw.dataimpulse.com:823

# país com exclusão de cidade
login__cr.br;nocity.saopaulo:senha@gw.dataimpulse.com:823


Dois avisos honestos: a seleção de país é obrigatória para usar cidade, estado, CEP ou ASN; e a DataImpulse não garante disponibilidade de uma cidade específica em qualquer momento — as mais populares têm boa cobertura, mas um pico de demanda pode empurrar seu request para outra região.

Traduzindo para orçamento: se o seu caso é "preciso que o IP seja brasileiro", você fica em US$ 1/GB. Se o caso exige "preciso que seja de São Paulo capital", o custo efetivo dobra. Vale a pena rodar 100–200 requests de teste na cidade-alvo antes de assumir um volume grande.

## Todos os planos, sem assinatura

A DataImpulse trabalha em pay-as-you-go. Não existe mensalidade, não existe renovação automática e o tráfego comprado não expira — GB não usado fica no saldo. A entrada mínima é US$ 5 em qualquer um dos quatro tipos de proxy.

| Tipo de proxy | Plano | Tráfego | Preço | Preço por GB | Link |
| --- | --- | --- | --- | --- | --- |
| Residencial | Intro | 5 GB | US$ 5 | US$ 1,00 | [Começar](https://bit.ly/dataimPulse) |
| Residencial | Basic | 50 GB | US$ 50 | US$ 1,00 | [Ver plano](https://bit.ly/dataimPulse) |
| Residencial | Advanced | 1 TB | US$ 800 | US$ 0,80 (20% off) | [Ver plano](https://bit.ly/dataimPulse) |
| Residencial | Custom | 5 TB+ | a partir de US$ 4.000 | negociável | [Falar com o time](https://bit.ly/dataimPulse) |
| Datacenter | Intro | 10 GB | US$ 5 | US$ 0,50 | [Começar](https://bit.ly/dataimPulse) |
| Datacenter | Basic | 100 GB | US$ 50 | US$ 0,50 | [Ver plano](https://bit.ly/dataimPulse) |
| Datacenter | Advanced | 1 TB | US$ 450 | US$ 0,45 | [Ver plano](https://bit.ly/dataimPulse) |
| Datacenter | Custom | 5 TB+ | a partir de US$ 2.250 | negociável | [Falar com o time](https://bit.ly/dataimPulse) |
| Mobile | Intro | 2,5 GB | US$ 5 | US$ 2,00 | [Começar](https://bit.ly/dataimPulse) |
| Mobile | Basic | 25 GB | US$ 50 | US$ 2,00 | [Ver plano](https://bit.ly/dataimPulse) |
| Mobile | Advanced | 1 TB | US$ 1.600 | US$ 1,60 | [Ver plano](https://bit.ly/dataimPulse) |
| Mobile | Custom | 5 TB+ | a partir de US$ 8.000 | negociável | [Falar com o time](https://bit.ly/dataimPulse) |
| Residencial Premium | Intro | 1 GB | US$ 5 | US$ 5,00 | [Testar pool premium](https://dataimpulse.com/pt/proxies-residenciais-premium/?aff=86938) |
| Residencial Premium | Basic | 10 GB | US$ 50 | US$ 5,00 | [Ver plano premium](https://dataimpulse.com/pt/proxies-residenciais-premium/?aff=86938) |
| Residencial Premium | Custom | 1 TB+ | a partir de US$ 4.000 | negociável | [Ver plano premium](https://dataimpulse.com/pt/proxies-residenciais-premium/?aff=86938) |

Repare na forma da curva de preço residencial: ela é quase plana. De 5 GB até cerca de 700–850 GB, a tarifa continua US$ 1/GB. O único degrau de desconto real aparece em 1 TB, quando cai para US$ 0,80/GB. Comprar volume "no meio do caminho" não rende nada além de uma fatura maior — não há motivo para comprar mais do que você consome no curto prazo, justamente porque o saldo não vence.

O plano residencial premium é outro produto, não uma versão melhor do mesmo: IPs de latência mais baixa, segmentação completa sem sobretaxa (inclusive cidade e ASN) e um gerente de conta dedicado. A US$ 5/GB, ele faz sentido quando a tarefa é crítica e o volume é pequeno. Para coleta de dados em escala média, é dinheiro jogado fora.

## Como conectar: gateway, portas e sessão sticky

A configuração é a mesma em qualquer linguagem. O endpoint é `gw.dataimpulse.com`, com portas diferentes conforme o comportamento de rotação:

- **823** — HTTP/HTTPS com rotação a cada request
- **824** — SOCKS5 com rotação a cada request
- **10000 a 20000** — portas de sessão sticky

Exemplo com cURL, forçando saída pelo Brasil em modo rotativo:

bash
curl -x "http://seulogin__cr.br:suaSenha@gw.dataimpulse.com:823" \
  https://api.ipify.org/


Em Python, o mesmo endereço entra no parâmetro `proxies`:

python
import requests

proxy = "http://seulogin__cr.br:suaSenha@gw.dataimpulse.com:823"
r = requests.get("https://httpbin.org/ip", proxies={"http": proxy, "https": proxy}, timeout=15)
print(r.json())  # confirme o IP de saída antes de rodar o job


Dois detalhes que economizam tempo de depuração:

1. **Teste o IP de saída antes de rodar o scraper.** Um `curl` para `api.ipify.org` confirma país e operadora em segundos. Erro de sintaxe no parâmetro `cr.br` é a causa mais comum de tráfego saindo do país errado — e você paga por ele do mesmo jeito.
2. **Autenticação via usuário/senha ou whitelist de IP.** A whitelist é mais prática em servidor com IP fixo; usuário/senha é obrigatório quando o IP de origem varia.

## Rotativo ou sticky: escolha pelo alvo, não pelo gosto

O modo rotativo troca de IP a cada request. É o padrão para crawler de alto volume: distribui as requisições entre milhares de endereços e reduz a chance de um único IP acumular bloqueios.

O modo sticky mantém o mesmo IP amarrado a uma porta por um período definido. A configuração aceita de 1 a 120 minutos, com média real de cerca de 30 minutos. E aqui vale a explicação técnica que o suporte da DataImpulse dá abertamente: como os IPs vêm de usuários reais, se aquele dispositivo sai da rede, a sessão gira automaticamente para o próximo IP disponível. Você configura o intervalo desejado, mas a duração efetiva depende do aparelho do outro lado. Isso não é falha do provedor — é como funciona qualquer pool residencial baseado em consentimento.

Na prática:

- **SERP, preço, listagem pública, verificação de anúncio** → rotativo
- **Login, carrinho, fluxo com múltiplos passos, teste de checkout** → sticky, começando em 30 minutos

👉 [Criar conta e testar com os US$ 5 iniciais, sem assinatura](https://bit.ly/dataimPulse)

## Limites, reembolso e o que os testes de terceiros mostram

Nem tudo é vantagem, e vale listar o que atrapalha.

**Não existe teste gratuito.** Toda avaliação começa com uma compra de no mínimo US$ 5. Em compensação, o crédito não expira, então o custo de testar é US$ 5 em dinheiro real, e não uma amostra grátis com prazo de validade.

**Reembolso de 7 dias só em pagamento por cartão.** A garantia cobre planos Intro quando menos de 80% do tráfego foi consumido. Compra em cripto no plano Intro não é reembolsável — isso está documentado e vale ler antes de pagar em USDT.

**Segmentação avançada consome o dobro.** Já tratado acima, mas é o principal item que estoura orçamento de quem não leu a documentação.

**Sem PayPal em toda a documentação recente de checkout.** Os métodos confirmados incluem cartão via Stripe (Visa e Mastercard) e cripto via Cryptomus (Bitcoin, Ethereum, USDT, Litecoin), com PayPal, transferência bancária, Alipay, Apple Pay e Google Pay aparecendo com disponibilidade variável por região. Se PayPal é a única forma de pagamento aprovada na sua empresa, confirme antes.

Do lado externo, o que existe de avaliação independente:

- A TechRadar, em análise própria, reportou taxa de sucesso consistentemente alta em scraping com os proxies residenciais.
- A HostAdvice deu nota geral 9,1/10, com 9,5/10 em preço, e registrou resposta humana no chat em cerca de 7 minutos.
- A Proxyway concedeu o prêmio de Maior Progresso.
- Levantamentos de terceiros apontam nota na casa de 4,6/5 no Trustpilot e 4,8/5 no G2.

E o contraponto, que também aparece em análises independentes: o pool é mais raso em geografias de terceiro nível (África subsaariana, Ásia Central) do que Bright Data e Oxylabs, e alvos protegidos por Cloudflare têm taxa de sucesso menor do que em boutiques especializadas. Para o Brasil, que está entre os mercados principais, esse ponto pesa menos — mas pesa.

## Comparando com o mercado antes de decidir

O US$ 1/GB é agressivo, mas não é o menor preço absoluto do mercado em todos os volumes. Comparações publicadas mostram que, entre 100 e 150 GB, provedores como a Evomi trabalham na faixa de US$ 0,49/GB — metade do preço — enquanto a DataImpulse continua em US$ 1/GB até o degrau de 1 TB. Em 1 TB, a Evomi também sai mais barata no residencial rotativo.

Então onde está a vantagem real?

Uma análise de custo da webscraping.ai coloca o ponto de equilíbrio em torno de 50 GB/mês: abaixo disso, o modelo pay-as-you-go de US$ 1/GB com mínimo de US$ 5 ganha de qualquer assinatura, porque assinatura cobra por um pacote que você não termina de usar. Acima disso, principalmente passando de 100 GB, provedores com tarifa decrescente por volume ficam mais baratos por gigabyte.

O segundo diferencial é o tráfego que não expira. Se o seu projeto roda em picos — um scraping grande em março, outro em agosto — você compra GB e usa quando quiser. Não há "use ou perca" no fim do ciclo mensal, e isso muda o cálculo para quem tem demanda irregular.

Se autenticação de contas em múltiplos perfis de navegador é o caso de uso principal, nenhum pool rotativo resolve bem. Para isso, proxies ISP estáticos são a ferramenta certa. Não é limitação da DataImpulse especificamente; é a natureza de IP residencial rotativo.

## Vale a pena para o seu caso?

Cenários em que o encaixe é bom:

- Você precisa de IP brasileiro, com volume abaixo de ~100 GB/mês
- O trabalho é intermitente e você não quer assinatura mensal
- Você quer conferir o tamanho do pool do Brasil antes de comprar, e não descobrir depois
- Você testa antes de escalar e prefere gastar US$ 5 a assinar um plano anual

Cenários em que procure outra coisa:

- Volume acima de 150–200 GB/mês com previsibilidade — tarifas decrescentes de concorrentes ganham
- Multi-conta em redes sociais ou e-commerce — precisa de ISP estático
- Alvos agressivos com Cloudflare pesado ou geografias fora do eixo principal
- Segmentação por cidade em volume alto — a sobretaxa de 2× encarece rápido

## Perguntas frequentes

**Existem IPs residenciais brasileiros de verdade no pool?**
Sim. O contador público da própria empresa mostra na casa de 41 mil IPs ativos no Brasil e mais de 580 mil IPs únicos em 30 dias. A DataImpulse mantém cluster próprio na região e adicionou novos endereços IP de entrada em atualização de infraestrutura divulgada no blog.

**Quanto custa 1 GB de proxy residencial brasileiro?**
US$ 1/GB quando você segmenta apenas por país. O tráfego roteado por filtros de cidade, estado, CEP ou ASN específico é cobrado em dobro, ficando perto de US$ 2/GB. O mínimo de entrada é US$ 5 (5 GB).

**O tráfego expira?**
Não. Esse é um dos diferenciais centrais do modelo: você compra GB, o saldo fica na conta e é consumido quando o projeto rodar.

**Dá para manter o mesmo IP brasileiro por muito tempo?**
Sessões sticky são configuráveis de 1 a 120 minutos em portas de 10000 a 20000, com média efetiva perto de 30 minutos. Como os IPs vêm de usuários reais, a sessão gira antes se o dispositivo sair da rede.

**Existe teste grátis?**
Não há período gratuito. O caminho é o plano Intro de US$ 5, que inclui 5 GB residenciais, não expira e tem garantia de reembolso de 7 dias em compras no cartão, desde que menos de 80% do tráfego tenha sido usado.

**Preciso de cartão internacional?**
O checkout aceita cartão Visa e Mastercard via Stripe e também cripto (Bitcoin, Ethereum, USDT, Litecoin) via Cryptomus. Outros métodos aparecem com disponibilidade variável por região, então vale confirmar no momento da compra.

## O resumo que interessa

IP brasileiro confiável para scraping, monitoramento de SERP e verificação de anúncio custa, hoje, US$ 1/GB na DataImpulse — sem mensalidade, sem expiração de saldo e com o tamanho do pool nacional publicado para você checar antes. O que precisa entrar na sua conta é a sobretaxa de 2× para segmentação por cidade e o fato de que, acima de 100–150 GB/mês, concorrentes com tarifa decrescente passam à frente no preço.

Se o seu perfil é o de volume baixo a médio, com demanda irregular e necessidade real de IPs do Brasil, o risco de testar é de cinco dólares.

👉 [Ver planos e preços atualizados da DataImpulse](https://bit.ly/dataimPulse)
