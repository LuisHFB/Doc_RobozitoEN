# Corte a Laser

Os cortes são feitos usando uma máquina de corte a laser e uma chapa de 300x300 mm.

[Arquivo DXF](docs/robozito_laser.dxf)

Caso queira fazer modificações ao projeto, seguem alguns guidelines para gerar os arquivos para corte.

!!! example "Máquina"

    === "Epilog"

        _**Arquivo PDF**_:

         - Linhas devem estar vetorizadas.
         - Linhas de corte com **0,025mm** de espessura.
         - Cor de linha preta e sem desfoque ou opacidade.
         - Gerar documento do tamanho do material que vai usar.
         
         
    === "DuploTech"

        _**Arquivo DXF**_:

        - Importar arquivos DXF para o [RDWorks](http://www.duploj.com.br/suporte).
        - Setar parametros de velocídade e potencia para cada cor.
        - Clicar em **"Salvar U file"** para exportar para o pendrive
