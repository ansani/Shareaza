# Shareaza Architecture

## 1. Overview

Shareaza is a peer-to-peer file-sharing application for Windows. Its architecture appears to be modular, with distinct components responsible for different aspects of its functionality. The application is built using C++ and utilizes COM for inter-component communication. Key architectural features include:

*   **Modular Design:** The codebase is organized into components with specific responsibilities, such as network communication, downloading, uploading, file management, search, user interface, and more.
*   **Multi-Network Support:** Shareaza supports multiple P2P networks, including Gnutella, Gnutella2 (G2), eDonkey, and BitTorrent, as evidenced by various constants and resource strings.
*   **Network Abstraction:** A connection component handles low-level network operations, supporting multiple protocols and both IPv4 and IPv6.
*   **File Management:** A library component manages shared and downloaded files, providing features like searching, hashing, and metadata management.
*   **User Interface:** A dedicated UI component (likely using MFC, based on `CWinApp` and resource definitions) handles user interaction.
*   **Extensibility:** The use of COM interfaces suggests that the application might support plugins or extensions. Resource files also indicate plugin-related settings.
*   **Integrated Features:** Shareaza includes additional integrated features such as a media player and an IRC client.

## 2. Core Components

Based on the analysis of header files, resource definitions, and class names, the following are some of the core components of Shareaza:

*   **Application (`CShareazaApp` / `CApplication`)**:
    *   **Responsibilities:**
        *   Acts as the main entry point and controller for the application (inherits from `CWinApp`).
        *   Manages application-level logic, including versioning, command-line parsing, and global settings.
        *   Initializes and provides access to other core components (e.g., UI, Library, Settings, GeoIP).
        *   Handles COM object creation and management.
        *   Manages global resources like fonts and themes.
        *   Provides logging and messaging capabilities.
*   **Connection (`CConnection`)**:
    *   **Responsibilities:**
        *   Manages network connections (sockets) with remote peers across different P2P networks.
        *   Handles sending and receiving data over the network.
        *   Measures and potentially throttles bandwidth usage.
        *   Detects the communication protocol being used by peers.
        *   Supports both IPv4 and IPv6 network protocols.
        *   Provides SSL/TLS encryption for secure communication.
*   **Download (`CDownload`, managed by `CDownloads`)**:
    *   **Responsibilities:**
        *   `CDownload`: Represents an individual file download. Manages its state (e.g., paused, active, completed), sources, progress, and interaction with the file system.
        *   `CDownloads` (implied, likely a manager class): Oversees all active and pending downloads.
        *   Allows users to pause, resume, cancel, and prioritize downloads.
        *   Handles file fragmentation and reassembly.
        *   Supports downloading via torrents (e.g., `IDD_TORRENT_GENERAL`, `IDD_TORRENT_FILES`).
        *   Tracks download progress, speed, and status.
        *   Interacts with a task manager (`CDownloadTask`) for specific download operations.
        *   Manages download groups and source selection.
        *   The UI is managed by `CDownloadsWnd` and related controls like `CDownloadsCtrl`.
*   **Upload (`CUploads`, with `CUploadTransfer`, `CUploadQueue`, `CUploadFile`)**:
    *   **Responsibilities:**
        *   `CUploads`: The central manager for upload functionality. Handles accepting incoming connections for uploads, managing active upload slots, and overall upload bandwidth.
        *   `CUploadQueue`: Manages the queue of files or parts of files waiting to be uploaded to specific peers.
        *   `CUploadFile`: Represents an individual file (or part of it) being prepared or currently uploading.
        *   `CUploadTransfer`: Represents an active upload session to a peer, handling the actual data transfer.
        *   Handles requests from peers for shared files.
        *   Tracks upload progress and status.
        *   Interacts with the Library component to access shared files.
        *   The UI is managed by `CUploadsWnd` and related controls like `CUploadsCtrl` and `CHomeUploadsBox`.
*   **Library (`CLibrary`)**:
    *   **Responsibilities:**
        *   Manages the user's collection of shared files and completed downloads.
        *   Scans specified folders for shareable files.
        *   Maintains a database of file information, including hashes (SHA1, TTH, MD5, eD2k), metadata, and availability.
        *   Provides local file searching capabilities.
        *   Monitors file changes and updates the library accordingly.
        *   Handles file sharing permissions and properties.
        *   Supports collections and album views.
        *   The UI is managed by `CLibraryWnd` and related controls.
*   **Search (`CSearchWnd`, `CQuerySearch`, `CManagedSearch`)**:
    *   **Responsibilities:**
        *   `CSearchWnd`: The main UI coordinator for search operations. Manages search tabs/windows and displays aggregated results.
        *   `CQuerySearch`: Represents the search query parameters (keywords, filters, network scope).
        *   `CManagedSearch`: Manages the lifecycle of an individual search operation (starting, stopping, monitoring).
        *   Handles searching for files across the supported P2P networks.
        *   Displays search results from various networks, likely using `CSearchMatch` or `CMatchFile` to represent individual results.
        *   Allows users to initiate downloads from search results.
        *   Interacts with the Connection component to send search requests and receive responses.
        *   UI elements include `CSearchPanel` for input, `CSearchDetailPanel` for result details, and `CSearchMonitorWnd` for tracking active searches.
*   **User Interface (MFC based, coordinated by `CMainWnd` and various `C...Wnd` classes)**:
    *   **Responsibilities:**
        *   Provides the graphical user interface for interacting with the application.
        *   `CMainWnd` (typically the main frame in an MFC app) hosts various views.
        *   Specialized window classes like `CDownloadsWnd`, `CUploadsWnd`, `CLibraryWnd`, `CSearchWnd`, `CNetWnd` (for Network view), `CMediaWnd` (for Media Player), `CIrcWnd` (for IRC) manage their respective UI areas.
        *   Displays information about downloads, uploads, library, search results, network status, etc.
        *   Allows users to configure extensive application settings through various dialogs (`IDD_SETTINGS_*`).
        *   Handles user input and commands, delegating actions to backend components.
        *   Supports skinning for UI customization (`IDD_SETTINGS_SKINS`).
*   **Settings Management (Likely a dedicated component or integrated within `CShareazaApp`)**:
    *   **Responsibilities:**
        *   Manages and persists user-configurable settings for all aspects of the application (Connection, Downloads, Uploads, Library, Networks, UI, etc.).
        *   Provides an interface for other components to access and modify settings.
*   **Networking (A collection of classes and managers for different P2P protocols)**:
    *   **Responsibilities:**
        *   Manages connections and communication for specific P2P networks (Gnutella, G2, eDonkey, BitTorrent, Discovery services like Kad).
        *   Includes classes like `CGnutella`, `CEDonkey`, `CBitTorrent` (these are inferred names based on common patterns and resource strings like `IDD_SETTINGS_GNUTELLA`, `IDD_SETTINGS_DONKEY`, `IDD_SETTINGS_BITTORRENT`).
        *   Handles protocol-specific logic for each network (e.g., handshakes, packet formats, peer communication).
        *   Manages peer lists, host caches (`IDR_HOSTCACHEFRAME`), and discovery services (`IDR_DISCOVERYFRAME`).
*   **Media Player (`CMediaWnd` and related classes)**:
    *   **Responsibilities:**
        *   Allows users to play audio and video files from the library or downloads.
        *   Provides standard media playback controls (play, pause, stop, seek, volume).
        *   Manages playlists and media information.
        *   May support visualizations (`IDD_MEDIA_VIS`).
*   **IRC Client (`CIrcWnd` and related classes)**:
    *   **Responsibilities:**
        *   Provides an integrated IRC client for chatting on IRC networks.
        *   Manages IRC server connections, channels, and user interactions.
        *   Supports common IRC commands and features.

## 3. Component Interactions

The core components of Shareaza interact in a complex but coordinated manner:

*   **`CShareazaApp`** acts as the central orchestrator, initializing and providing access to other components. UI actions, often originating from `C...Wnd` classes, would trigger calls to the appropriate backend components, sometimes routed via `CShareazaApp` or directly if components have handles to each other.
*   **`CDownloads`**, **`CUploads`**, and **Search components (`CSearchWnd`, `CManagedSearch`)** heavily rely on the **Networking components** (including `CConnection` and protocol-specific managers) to establish connections with peers, transfer data, and send/receive search queries/results across different P2P networks.
*   **`CDownloads`** interacts with **`CLibrary`** to add completed downloads, check for existing files, and potentially update file metadata. It uses the **Networking components** to find sources and download data.
*   **`CUploads`** interacts with **`CLibrary`** to access shared files and their metadata, and with **Networking components** to serve files to other peers.
*   **Search components** use **Networking components** to broadcast search queries and gather results. Results are then displayed in the **`CSearchWnd`**, and initiating a download from a search result would involve the **`CDownloads`** component.
*   **`CLibrary`** may interact with **Networking components** to announce shared files or respond to queries about local files from other peers. It also interacts with the **User Interface (`CLibraryWnd`)** to display file lists, folder structures, and metadata.
*   The **Media Player (`CMediaWnd`)** and **IRC Client (`CIrcWnd`)** are integrated into the **User Interface** and would interact with `CLibrary` (for media files) and **Networking components** (for IRC) respectively.
*   All components likely interact with the **Settings Management** system to retrieve and store their configurations.

## 4. Further Investigation Needed

*   Confirm the specific class names for the per-network managers (e.g., for Gnutella, eDonkey, BitTorrent) by examining their respective settings dialogs' source code or related `.cpp` files.
*   Delve deeper into the `shareaza/*.cpp` files to understand the implementation details of each component and their precise interactions (e.g., how `CUploads` uses `CUploadQueue` and `CUploadTransfer`).
*   Analyze the threading model to understand how Shareaza manages concurrent operations (downloads, uploads, network communication, UI updates).
*   Investigate the plugin architecture in more detail, including how plugins are loaded and how they interact with the core application using the defined COM interfaces.
*   Map out the specific classes and mechanisms used for each supported P2P network protocol in more detail.

---

This document provides an updated overview of the Shareaza architecture based on header files, resource file analysis, and `grep` results for class names. Further in-depth code review is necessary for a complete and detailed understanding.
