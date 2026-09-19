# Konduto
Uma linguagem para quem quer especificar O QUE fazer, sem se preocupar com COMO será feito.

**ATENÇÃO**: Em Fase Experimental. A sintaxe e os conceitos podem mudar.

## 1. Introdução

Se você der o mesmo requisito a 10 programadores, provavelmente você vai obter 10 programas diferentes.

Cada programador tem seu jeito de programar, seus padrões de projeto preferidos, suas prioridades, seu estilo, etc.

Normalmente, essas diferenças não geram prejuízo em projetos pequenos ou simples.

Entretanto, quando se trata de projetos duradouros e complexos, esses diversos “jeitos” de programar podem causar uma redução na produtividade da equipe, pois, além do tempo necessário para implementar o que foi pedido, o programador começa a gastar cada vez mais tempo tentando entender o código-fonte que já existe.

E a maioria absoluta dessas diferenças não agregam valor para o usuário. E, mesmo quando há um ganho de desempenho, normalmente ele é imperceptível para quem usa o sistema.

Essa é a motivação para a criação da Konduto, uma linguagem focada em especificar aquilo que importa para o usuário: o comportamento do sistema.

A palavra “konduto” significa “comportamento” em Esperanto.

## 2. Exemplo

O trecho a seguir mostra o que seria necessário escrever em Konduto para escrever um sistema chamado "Exemplo", que possui uma entidade "Pessoa" e uma única funcionalidade chamada "Cadastrar Pessoa":

```json
{
  "nome": "Exemplo",
  "entidades": [
    {
      "nome": "Pessoa",
      "atributos": [
        {
          "nome": "Nome",
          "tipo": "Texto"
        }
      ]
    }
  ],
  "funcionalidades": [
    {
      "nome": "Cadastrar Pessoa",
      "componentes": [
        {
          "campoTextoSimples" {
            "nome": "Nome"
          }
        },
        {
          "botao" {
            "nome": "Salvar",
            "aoClicar": "[exibido a seguir]"
          }
        }
      ],
      "aoIniciar": "exibe campos 'Nome','Salvar';"
    }
  ]
}
```

O trecho a seguir é o conteúdo do atributo "aoClicar", o qual indica as ações que devem ser realizadas ao clicar no botão Salvar:

```
declara 'p' do tipo 'Pessoa';
atribui a (p) o valor (nova('Pessoa));
atribui a (p.nome) o valor (nome);
salva (p);
```

## 3. Especificação

Para saber a especificação completa da linguagem Konduto [clique aqui](ESPECIFICACAO.md).


