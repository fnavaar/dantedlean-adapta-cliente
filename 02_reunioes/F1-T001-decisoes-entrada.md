# Decisões F1-T001 — Contrato de entrada, campos, papéis e fixture

> **Registrado em:** 2026-08-26 · **Por:** Dante (Champion/Cliente) · **Aprovado por:** Navaar (Consultor)

## 1. Fonte oficial dos pedidos

- **Fonte:** MaxiProd (ERP)
- **Tipo:** API ou exportação do MaxiProd
- **Nota:**Pedidos são gerados no MaxiProd e entram automaticamente na fila do Compass para programação.

## 2. Campos mínimos obrigatórios

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| Número do pedido | string | sim | Identificador externo do MaxiProd |
| Cliente | string | sim | Nome/razão social do cliente |
| Data prometida | date | sim | Data de entrega prometida |
| Lista de produtos | array | sim | Produtos do pedido (mínimo 1 item) |
| Local de entrega | string | sim | Endereço/destino da entrega |
| Quantidade de itens | number | sim | Total de itens no pedido |

**Contrato canônico:** `order_external_id`, `customer`, `promised_date`, `products[]`, `delivery_location`, `item_count`.

## 3. Papéis e permissões

| Papel | Quem | Permissões |
|---|---|---|
| Demanda (criação) | Comercial | Cria/edita demanda em rascunho |
| Elegibilidade | **Automática** | Após inserção do PV no MaxiProd, o sistema valida automaticamente |
| Visibilidade | Admin e Gestores | Veem todas as demandas |
| Visibilidade | Demais usuários | Veem apenas suas demandas |

**Nota:** A validação de elegibilidade é automática — não requer aprovação manual do PCP.

## 4. Fixture de teste

- **Fonte:** Usar a fixture já preparada por Navaar
- **Responsável:** Navaar fornece o arquivo de teste

## 5. Pré-fluxo pedido → projeto

- Quando o PV é gerado no MaxiProd, ele entra **automaticamente** na fila do Compass para programação.
- Se o projeto já possui desenho padrão → o desenho é utilizado automaticamente.
- Se é um novo projeto → deve ser projetado pela Engenharia (fluxo da SPEC-1-002).

---

**Status:** ✅ Decisões aprovadas pelo Champion (Dante) e Registrador pelo Consultor (Navaar).