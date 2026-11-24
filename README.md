"""
================================================================================
ÁRVORE DE DECISÃO INTERATIVA E PROGRAMÁTICA PARA SELEÇÃO DO MÉTODO DE ICV
================================================================================
Métodos Suportados: 
  - PLCA (Process-Based Life Cycle Assessment)
  - EIO-LCA (Economic Input-Output Life Cycle Assessment)
  - HLCA (Integrated Hybrid Life Cycle Assessment)
  - PLCA_IMPUTATION (PLCA com Preenchimento de Lacunas / Surrogates)

Grounding: 
  - Projeto de Pesquisa: Modelagem e Análise de Inventários do Ciclo de Vida
  - Documento de Referência: 'Métodos de ICV'
================================================================================
"""

import argparse
from dataclasses import dataclass
from typing import List


@dataclass(frozen=True)
class MetodoICV:
    """Estrutura tipada para representar a recomendação de um método de ICV."""
    sigla: str
    nome: str
    abordagem: str
    equacao: str
    vantagens: List[str]
    limitacoes: List[str]


def classificar_caso_estudo(
    especificidade_fisica: bool,
    erro_truncamento_aceitavel: bool,
    capacidade_hibridizacao: bool
) -> MetodoICV:
    """
    Classifica programaticamente o método de ICV ideal com base nos critérios do projeto.

    Parâmetros:
      - especificidade_fisica (bool): True se o estudo exige detalhamento de processos
        físicos e bens heterogêneos.
      - erro_truncamento_aceitavel (bool): True se omitir 20% a 50%+ de impactos indiretos
        é aceitável para o escopo.
      - capacidade_hibridizacao (bool): True se a equipe possui recursos para construir
        matrizes Cu/Cd e depurar dupla contagem.
    """
    if not especificidade_fisica:
        return MetodoICV(
            sigla="EIO-LCA",
            nome="ACV Econômica / Insumo-Produto (EIO-LCA)",
            abordagem="Top-down macroeconômica",
            equacao="q_s = R * (I - A)^(-1) * y_s",
            vantagens=[
                "Garante completude sistêmica na fronteira (infinito alcance)",
                "Elimina totalmente o erro de truncamento a montante"
            ],
            limitacoes=[
                "Elevada agregação setorial (não diferencia produtos do mesmo setor)",
                "Premissa de linearidade monetário-física e defasagem temporal de tabelas IO"
            ]
        )

    if erro_truncamento_aceitavel:
        return MetodoICV(
            sigla="PLCA",
            nome="ACV Baseada em Processo (PLCA / Process-Based LCA)",
            abordagem="Bottom-up física",
            equacao="q_u = R_p * T^(-1) * y_u",
            vantagens=[
                "Máxima especificidade física e fidelidade técnica por unidade de processo",
                "Estrita conformidade com as normas ISO 14040 e ISO 14044"
            ],
            limitacoes=[
                "Erro de truncamento sistemático (omissão de 20% a mais de 100% dos impactos indiretos)",
                "Elevada demanda de tempo e custo para coleta de dados primários"
            ]
        )

    if capacidade_hibridizacao:
        return MetodoICV(
            sigla="HLCA",
            nome="ACV Híbrida Integrada (HLCA / Integrated Hybrid Analysis)",
            abordagem="Integração Foreground Físico (PLCA) + Background Econômico (EIO-LCA)",
            equacao="q_h = [R_p  0; 0  R] * [T  -C_u; -C_d  I-A]^(-1) * [y_u; 0]",
            vantagens=[
                "Combina a precisão física do primeiro plano com a completude do segundo plano",
                "Elimina o erro de truncamento sem perder a granularidade dos processos-chave",
                "Facilidade estatisticamente superior na interpretação ambiental final (Bloco E: Média = 2.20, p = 0.0348)"
            ],
            limitacoes=[
                "Complexidade matemática na montagem das matrizes de corte (C_u, C_d) e acoplamento (Bloco D)",
                "Necessidade de algoritmos de depuração de dupla contagem"
            ]
        )

    return MetodoICV(
        sigla="PLCA_IMPUTATION",
        nome="PLCA com Preenchimento de Lacunas (Selection Proxy / Data-Driven)",
        abordagem="Bottom-up física assistida por modelos de imputação",
        equacao="q_u = R_p * T^(-1) * y_u + Surrogates",
        vantagens=[
            "Mitiga lacunas de dados sem incorrer no elevado custo matricial do HLCA",
            "Mantém a estrutura física da norma ISO 14040/14044"
        ],
        limitacoes=[
            "Incerteza associada ao uso de inventários substitutos (surrogates)",
            "Persistência parcial do truncamento de fronteira"
        ]
    )


def ler_resposta_sim_nao(prompt: str = "   [S]im / [N]ão: ") -> bool:
    """
    Lê e valida a entrada do usuário no terminal, aceitando apenas variações de Sim ou Não.
    """
    while True:
        resp = input(prompt).strip().upper()
        if resp in ("S", "SIM"):
            return True
        if resp in ("N", "NAO", "NÃO"):
            return False
        print("   ⚠ Entrada inválida. Digite apenas 'S' para Sim ou 'N' para Não.")


def exibir_resultado(res: MetodoICV) -> None:
    """Formata e imprime o resultado da classificação no terminal."""
    print("\n" + "=" * 82)
    print(f" RECOMENDAÇÃO FINAL: {res.nome}")
    print("=" * 82)
    print(f"• Sigla: {res.sigla}")
    print(f"• Abordagem: {res.abordagem}")
    print(f"• Formulação Matemático-Matricial: {res.equacao}\n")
    print("✔ VANTAGENS CHAVE:")
    for vantagem in res.vantagens:
        print(f"   - {vantagem}")
    print("\n⚠ ATENÇÃO E LIMITAÇÕES:")
    for limitacao in res.limitacoes:
        print(f"   - {limitacao}")
    print("=" * 82 + "\n")


def executar_arvore_interativa() -> None:
    """Executa a árvore de decisão via terminal de forma interativa (CLI)."""
    print("=" * 82)
    print("  SISTEMA DE APOIO À DECISÃO: SELEÇÃO DE MÉTODOS DE INVENTÁRIO DO CICLO DE VIDA (ICV)")
    print("=" * 82)
    print("Responda às perguntas para determinar a abordagem recomendada (PLCA, EIO-LCA ou HLCA).\n")

    # Pergunta 1
    print("1. O objetivo do estudo exige especificidade física de processos industriais")
    print("   e diferenciação técnica de bens heterogêneos no mesmo setor?")
    esp_fisica = ler_resposta_sim_nao()

    if not esp_fisica:
        exibir_resultado(classificar_caso_estudo(False, False, False))
        return

    # Pergunta 2
    print("\n2. O erro de truncamento de fronteira (omissão de 20% a 50%+ dos impactos indiretos)")
    print("   é ACEITÁVEL para o escopo, OU a cadeia de suprimentos física está 100% mapeada?")
    trunc_aceitavel = ler_resposta_sim_nao()

    if trunc_aceitavel:
        exibir_resultado(classificar_caso_estudo(True, True, False))
        return

    # Pergunta 3
    print("\n3. A equipe possui recursos computacionais, tempo e capacidade matemática para")
    print("   construir matrizes de acoplamento (C_u, C_d, preços) e tratar a dupla contagem?")
    cap_hibrida = ler_resposta_sim_nao()

    exibir_resultado(classificar_caso_estudo(True, False, cap_hibrida))


def executar_demonstracao() -> None:
    """Executa uma bateria automatizada de exemplos cobrindo todos os ramos da árvore."""
    print("=== DEMONSTRAÇÃO AUTOMATIZADA DA ÁRVORE DE DECISÃO ===\n")
    casos_teste = [
        ("Estudo de Triagem Macroeconômica Setorial", False, False, False),
        ("ACV de Processo com Dados Primários Completos", True, True, False),
        ("Ecodesign Completo sem Truncamento (Com Recursos Matemáticos)", True, False, True),
        ("Ecodesign sem Recursos para Acoplamento Híbrido Complexo", True, False, False),
    ]
    for desc, p1, p2, p3 in casos_teste:
        r = classificar_caso_estudo(p1, p2, p3)
        print(f"📌 {desc}")
        print(f"   ► Recomendado: {r.sigla} ({r.nome})")
        print(f"   ► Equação: {r.equacao}\n")
    print("Dica: Para executar a versão interativa no terminal, rode: python3 arvore_decisao_icv.py --cli")


def main() -> None:
    parser = argparse.ArgumentParser(
        description="Árvore de Decisão para Seleção de Métodos de Inventário do Ciclo de Vida (ICV)."
    )
    parser.add_argument(
        "--cli",
        action="store_true",
        help="Executa o modo interativo via linha de comando."
    )
    args = parser.parse_args()

    if args.cli:
        executar_arvore_interativa()
    else:
        executar_demonstracao()


if __name__ == "__main__":
    main()
