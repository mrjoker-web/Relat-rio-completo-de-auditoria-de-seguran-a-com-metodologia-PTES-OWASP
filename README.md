<div align="center">

```
██████╗ ███████╗██╗      █████╗ ████████╗ ██████╗ ██████╗ ██╗ ██████╗
██╔══██╗██╔════╝██║     ██╔══██╗╚══██╔══╝██╔═══██╗██╔══██╗██║██╔═══██╗
██████╔╝█████╗  ██║     ███████║   ██║   ██║   ██║██████╔╝██║██║   ██║
██╔══██╗██╔══╝  ██║     ██╔══██║   ██║   ██║   ██║██╔══██╗██║██║   ██║
██║  ██║███████╗███████╗██║  ██║   ██║   ╚██████╔╝██║  ██║██║╚██████╔╝
╚═╝  ╚═╝╚══════╝╚══════╝╚═╝  ╚═╝  ╚═╝    ╚═════╝ ╚═╝  ╚═╝╚═╝ ╚═════╝

██████╗ ███████╗███╗   ██╗████████╗███████╗███████╗████████╗
██╔══██╗██╔════╝████╗  ██║╚══██╔══╝██╔════╝██╔════╝╚══██╔══╝
██████╔╝█████╗  ██╔██╗ ██║   ██║   █████╗  ███████╗   ██║
██╔═══╝ ██╔══╝  ██║╚██╗██║   ██║   ██╔══╝  ╚════██║   ██║
██║     ███████╗██║ ╚████║   ██║   ███████╗███████║   ██║
╚═╝     ╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚══════╝╚══════╝   ╚═╝
```

### `// Penetration Test — Rede Interna Empresarial`

**Relatório completo de auditoria de segurança com metodologia PTES / OWASP**

[![Status](https://img.shields.io/badge/status-concluído-52c98b?style=for-the-badge)](.)
[![Metodologia](https://img.shields.io/badge/metodologia-PTES%20%2F%20OWASP-e05260?style=for-the-badge)](.)
[![Plataforma](https://img.shields.io/badge/plataforma-Android%20%2B%20Termux-3DDC84?style=for-the-badge&logo=android&logoColor=white)](.)
[![Shadow Suite](https://img.shields.io/badge/tools-Shadow%20Suite-f0b800?style=for-the-badge)](https://github.com/mrjoker-web)
[![License](https://img.shields.io/badge/license-MIT-3776AB?style=for-the-badge)](.)

</div>

---

## `> about`

Relatório de Penetration Test realizado à rede interna de uma empresa cliente, com **autorização formal prévia**, a 2 de Maio de 2026.

O que torna este pentest especial: foi conduzido **inteiramente a partir de um dispositivo Android** usando Termux, com as ferramentas da **Shadow Suite** desenvolvidas de raiz pelo auditor. Uma prova de que metodologia e conhecimento valem mais do que hardware.

```
Duração    : ~4 horas
Scope      : Rede interna 192.168.1.0/24
Tipo       : Grey Box
Ferramentas: Nmap · ShadowScanner · ShadowCVE · Shadow CLI
Plataforma : Android + Termux + Arch Linux (proot)
```

---

## `> findings summary`

```
┌──────────────────────────────────────────────────────────────────┐
│                    RESUMO DE FINDINGS                            │
├──────────────┬────────────────────────────────────────┬─────────┤
│  SEVERIDADE  │  FINDING                               │  CVSS   │
├──────────────┼────────────────────────────────────────┼─────────┤
│  🔴 HIGH     │  Switch gerido sem autenticação         │  7.5    │
│  🔴 HIGH     │  Impressora acessível sem login         │  7.2    │
│  🟠 MEDIUM   │  Câmaras IP com creds potencial default │  5.3    │
│  🔵 INFO     │  SSH ISP exposto (fora de scope)        │  N/A    │
└──────────────┴────────────────────────────────────────┴─────────┘

Total: 4 findings  |  2 HIGH  |  1 MEDIUM  |  1 INFO
```

---

## `> hosts discovered`

```
Rede scaneada: 192.168.1.0/24

192.168.1.199  →  TP-LINK HTTPD/1.0          (Câmara IP / NVR)
192.168.1.219  →  TP-LINK Web Switch          (Switch gerido)     ⚠ HIGH
192.168.1.224  →  Konica Minolta bizhub C452  (Impressora)        ⚠ HIGH
192.168.1.243  →  TP-LINK HTTPD/1.0          (Câmara IP)
192.168.1.254  →  Huawei WAP sshd 1.5        (ONT NOS)           ℹ FORA SCOPE
```

---

## `> f-001 — switch gerido`

```
ID       : F-001
IP       : 192.168.1.219
Serviço  : HTTP/HTTPS — TP-LINK Web Switch
CVSS     : 7.5 — HIGH
```

**Evidência:**
```bash
$ nmap -sV -p 80,443 192.168.1.219
80/tcp  open  http   Web Switch
443/tcp open  ssl/http  Liaison Exchange Commerce Suite
```

**Impacto:** Acesso à consola de administração do switch permite redireccionar tráfego, criar VLANs, fazer port mirroring e comprometer toda a segmentação de rede.

**Remediação:**
- Activar autenticação forte no painel web
- Restringir acesso por IP de gestão dedicado
- Desactivar HTTP — usar exclusivamente HTTPS
- Actualizar firmware para versão mais recente

---

## `> f-002 — impressora konica minolta`

```
ID       : F-002
IP       : 192.168.1.224
Serviços : 80, 443, 8080 (gSOAP 2.7), 8081 (OpenAPI)
CVSS     : 7.2 — HIGH
```

**Evidência:**
```bash
$ nmap -sV 192.168.1.224
8080/tcp open  soap  gSOAP 2.7
8081/tcp open  http  Konica Minolta bizhub C452 OpenAPI

# Acesso ao painel web sem autenticação
http://192.168.1.224/wcd/spa_n  →  Sessão "Público" activa
```

**Impacto:** Acesso a trabalhos de impressão anteriores (documentos sensíveis), impressão directa não autorizada, credenciais SMTP/LDAP expostas nas configurações, e potencial RCE via CVE-2020-15902 (gSOAP 2.7).

**Remediação:**
- Activar autenticação no painel web (desactivar sessão "Público")
- Actualizar firmware — gSOAP 2.7 tem vulnerabilidade conhecida
- Isolar a impressora em VLAN dedicada
- Rever credenciais SMTP/LDAP armazenadas

---

## `> f-003 — câmaras ip tp-link`

```
ID       : F-003
IPs      : 192.168.1.199 · 192.168.1.243
Serviço  : TP-LINK HTTPD/1.0
CVSS     : 5.3 — MEDIUM
```

**Evidência:**
```bash
$ nmap -sV -p 80,443 192.168.1.199
80/tcp  open  http   TP-LINK HTTPD/1.0
443/tcp open  https  TP-LINK HTTPD/1.0
# Redirect automático para HTTPS no acesso HTTP
```

**Impacto:** Acesso potencial ao feed de vídeo e configurações se credenciais default activas (admin/admin).

**Remediação:**
- Alterar credenciais default imediatamente
- Actualizar firmware dos dispositivos
- Isolar câmaras em VLAN sem acesso à rede corporativa

---

## `> f-004 — responsible disclosure (nos)`

```
ID       : F-004 (INFO — Fora de scope)
IP       : 192.168.1.254
Serviço  : Huawei WAP sshd 1.5 — porta 22
```

**Evidência:**
```bash
$ nmap -sV -p 22 192.168.1.254
22/tcp open  ssh  Huawei WAP sshd 1.5 (protocol 2.0)

# Banner expõe: "Politica de Seguranca NOS..."
# Autenticação NÃO tentada — teste interrompido imediatamente
```

**Acção tomada:** Identificado como equipamento NOS (fora de scope). Teste interrompido de imediato. Vulnerabilidade reportada à NOS via **Responsible Disclosure** (ISO/IEC 29147).

> *Este é o momento que separa um pentester ético de todo o resto — parar quando chegamos a um limite que não é nosso.*

---

## `> methodology`

```
FASES DO PENTEST (PTES)

  01  Reconhecimento
      └─ Nmap host discovery · 192.168.1.0/24

  02  Scanning & Enumeração
      └─ nmap -A -p- · ShadowScanner --top-ports · Banner grabbing

  03  Análise de Vulnerabilidades
      └─ ShadowCVE · Nikto · Manual analysis

  04  Exploração (verificação — não destrutiva)
      └─ Acesso ao painel web impressora (sem autenticação)
         Tentativa de login SSH (credenciais default — falhada)

  05  Relatório & Remediação
      └─ PDF profissional gerado com Shadow Suite
         Responsible Disclosure à NOS
```

---

## `> tools used`

| Ferramenta | Função | Repo |
|-----------|--------|------|
| 🛡️ ShadowScanner | Network scan + banner grabbing | [mrjoker-web/ShadowScanner](https://github.com/mrjoker-web/ShadowScanner) |
| 🔎 ShadowCVE | CVE lookup por serviço/versão | [mrjoker-web/ShadowCVE](https://github.com/mrjoker-web/ShadowCVE) |
| 🖥️ Shadow CLI | Pipeline recon completo | [mrjoker-web/ShadowCLI](https://github.com/mrjoker-web/ShadowCLI) |
| 🔍 Nmap 7.95 | Port scanning | nmap.org |

**Plataforma:** Android 13 · Termux · Arch Linux (proot) · Python 3

---

## `> report`

O relatório completo em PDF está disponível neste repositório:

📄 **[relatorio_pentest.pdf](./relatorio_pentest.pdf)**

```
Conteúdo do relatório:
  ├─ Capa profissional (tema dark · Shadow Suite)
  ├─ Sumário executivo
  ├─ Âmbito e metodologia
  ├─ Hosts descobertos
  ├─ 4 finding cards (CVSS · evidência · impacto · remediação)
  └─ Conclusão e plano de remediação por prioridade
```

---

## `> timeline`

```
[14:00] Início do scan — nmap 192.168.1.0/24
[14:18] Hosts identificados: 5 dispositivos activos
[14:35] ShadowScanner — deep scan a todos os hosts
[15:10] F-001 confirmado — switch sem auth
[15:24] F-002 confirmado — impressora painel público
[15:45] F-003 identificado — câmaras TP-LINK
[16:10] SSH 192.168.1.254 — banner NOS identificado
[16:12] Teste INTERROMPIDO — fora de scope
[17:00] Relatório escrito e entregue ao cliente
[17:30] Email Responsible Disclosure enviado à NOS
```

---

## `> key takeaways`

```
✅ Hardware caro não é necessário — Android + Termux é suficiente
✅ Metodologia estruturada > ferramentas caras
✅ Dispositivos IoT (câmaras, impressoras) são os alvos mais esquecidos
✅ Responsible Disclosure é obrigatório — sempre
✅ Documentação rigorosa diferencia amadores de profissionais
```

---

## `> shadow suite`

| Tool | Descrição | Repo |
|------|-----------|------|
| 🌐 ShadowSub | Subdomain finder | [mrjoker-web/ShadowSub](https://github.com/mrjoker-web/ShadowSub) |
| ⚡ ShadowProbe | HTTP/HTTPS probe | [mrjoker-web/ShadowProbe](https://github.com/mrjoker-web/ShadowProbe) |
| 🔍 ShadowScan | Recon tool | [mrjoker-web/ShadowScan-Tool](https://github.com/mrjoker-web/ShadowScan-Tool) |
| 🛡️ ShadowScanner | Network scanner | [mrjoker-web/ShadowScanner](https://github.com/mrjoker-web/ShadowScanner) |
| 📱 ShadowDroid | Android ADB audit | [mrjoker-web/ShadowDroid-](https://github.com/mrjoker-web/ShadowDroid-) |
| ⚙️ ShadowSetup | Terminal setup | [mrjoker-web/ShadowSetup](https://github.com/mrjoker-web/ShadowSetup) |
| 🖥️ Shadow CLI | Full recon framework | [mrjoker-web/ShadowCLI](https://github.com/mrjoker-web/ShadowCLI) |
| 🧪 Shadow Lab | Practice environment | [mrjoker-web/ShadowLab](https://github.com/mrjoker-web/ShadowLab) |
| 🔎 ShadowCVE | CVE Intelligence | [mrjoker-web/ShadowCVE](https://github.com/mrjoker-web/ShadowCVE) |
| 📖 Guia Pentest | Guia para aprendizes | [mrjoker-web/GUIA-PENTEST](https://github.com/mrjoker-web/GUIA-PENTEST) |

---

## `> disclaimer`

```
⚠  AVISO LEGAL

Este pentest foi realizado com autorização formal e escrita
da empresa cliente, em ambiente controlado.

Todos os testes foram conduzidos dentro do scope acordado.
Ao identificar equipamento fora do scope (NOS), o teste
foi imediatamente interrompido e reportado responsavelmente.

A reprodução destas técnicas sem autorização é ilegal —
Lei do Cibercrime n.º 109/2009 (Portugal / UE).
```

---

## `> author`

<div align="center">

Feito por **[Mr Joker](https://github.com/mrjoker-web)** — Aspiring Pentester & Python Tools Developer · Lisboa, PT

[![GitHub](https://img.shields.io/badge/GitHub-mrjoker--web-181717?style=for-the-badge&logo=github)](https://github.com/mrjoker-web)
[![Telegram](https://img.shields.io/badge/Telegram-mr__joker78-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/mr_joker78)
[![Twitter/X](https://img.shields.io/badge/X-mrjoker3790-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/mrjoker3790)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mr%20Joker-0e76a8?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/mr-joker-951ab2357)

*Se achares útil, deixa uma ⭐*

</div>
