
name: Run SDI Experiment

on:
  workflow_dispatch:    # Permet de lancer manuellement depuis GitHub

jobs:
  run-sdi:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Read One-Shot SDI instructions
        id: read_sdi
        run: |
          echo "content<<EOF" >> $GITHUB_OUTPUT
          cat experiments/sdi_oneshot.md >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Ask GitHub Copilot to run SDI experiment
        id: copilot
        uses: github/copilot-actions@v1
        with:
          instructions: |
            Tu es un agent IA.
            Exécute entièrement le protocole SDI fourni ci-dessous :
            ---
            ${{ steps.read_sdi.outputs.content }}
            ---
            Puis génère :
            1. Un fichier CSV complet
            2. Un rapport scientifique complet (Markdown)
            Donne la sortie sous forme de blocs de code.

      - name: Save CSV
        run: |
          mkdir -p results
          echo "${{ steps.copilot.outputs.csv }}" > results/sdi_results.csv

      - name: Save report
        run: |
          echo "${{ steps.copilot.outputs.report }}" > results/rapport_sdi.md

      - name: Upload results as build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: SDI-Results
          path: results/

