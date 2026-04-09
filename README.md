# 容器化與虛擬化技術

Containerization and Virtualization Technologies

## 課程簡介

本課程從虛擬機建置出發，逐步引導學生掌握虛擬化與容器化的核心技術，涵蓋 VM 環境建置、虛擬網路、多 VM 分層管理，以及 Docker 容器化應用等主題。

## 實驗單元

| 週次 | 主題 | 關鍵技術 |
|------|------|----------|
| W01 | 虛擬化概論、環境建置與 Snapshot 機制 | VMware、Ubuntu VM、Docker 安裝、Snapshot |
| W02 | VMware 網路模式與雙 VM 排錯 | NAT / Bridged / Host-only、網路診斷 |
| W03 | 多 VM 架構：分層管理、跳板機與最小暴露設計 | SSH 金鑰認證、ufw 防火牆、ProxyJump |
| W04 | Linux 系統基礎：檔案系統、權限、程序與服務管理 | FHS、chmod/chown、systemctl/journalctl、$PATH |
| W05 | 容器底層原理：Namespace、Cgroups、Union FS | /proc/\<pid\>/ns、cgroup v2、overlay2、OCI |
| W06 | Docker Image 與 Dockerfile | Dockerfile 指令、layer 快取、multi-stage build |
| 期中 | [期中實作：雙 VM 分層部署與故障排查](midterm_student.md) | 整合 W01–W06，約 2 小時 |

## 環境需求

- VMware Workstation Pro（Windows/Linux）或相容替代方案（Mac）
- Ubuntu Server VM
- Docker Engine

## 授權

本課程教材僅供課堂學習使用。
