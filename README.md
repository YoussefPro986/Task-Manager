# Task-Manager
Advanced Task Manager in MFC

This article demonstrates how to create a Task Manager in MFC which lists all the processes running in the system along with their process details and version information.

This article demonstrates how to create a Task Manager in MFC which lists all the processes running in the system
 along with their process details and version information.

Download Source Files - (22.8 Kb)

https://mega.nz/file/3fRSSboT#ivwdMdBtYdILJLjqkQrZt_HFOrCC_i0Y1d3w0pPyZFQ

Download Demo Project - (8.21 Kb)

https://mega.nz/file/vOBE3ZbR#2SVDzPsHWeU1HUDU_YNyx_WuJcNGCrPGDxLn1yHXMy4

Introduction
This article demonstrates how to create a task manager using MFC. This task manager known as Advanced Task Manager will display extra information than a standard task manager, like the properties of a running executable and process details.

Background
The standard task manager available will show the details about running processes, but in some cases, we need to know the path of the executable from which the process is running, which is not available. This application will show the details of a process like the path from which it is executing, product name, product version etc.

Implementation
The design is simple here. There are three classes which perform the action. CProcessEnumerator is responsible for loading all NT processes. All these processes are loaded to a list control. CProcessViewer can provide information about a specified process, and CVersionViewer can provide information about a running executable.

The following code enumerates all the running processes on a WIN NT/2000 system:

//
BOOL CProcessEnumerator::EnumWinNTProcs( PROCENUMPROC lpProc, LPARAM lParam )
{
    LPDWORD        lpdwPIDs ;
    DWORD          dwSize, dwSize2, dwIndex ;
    HMODULE        hMod ;
    HANDLE         hProcess ;
    char           szFileName[ MAX_PATH ] ;
    ENUMINFOSTRUCT sInfo ;

    dwSize2 = 256 * sizeof( DWORD ) ;
    lpdwPIDs = NULL ;
    do
    {
        if( lpdwPIDs )
        {
            HeapFree( GetProcessHeap(), 0, lpdwPIDs ) ;
            dwSize2 *= 2 ;
        }
        lpdwPIDs = (LPDWORD)HeapAlloc( GetProcessHeap(), 0, dwSize2 );
        if( lpdwPIDs == NULL )
        {
            return FALSE ;
        }
        if( !m_lpfEnumProcesses( lpdwPIDs, dwSize2, &dwSize ) )
        {
            HeapFree( GetProcessHeap(), 0, lpdwPIDs ) ;
            return FALSE ;
        }
    }while( dwSize == dwSize2 ) ;
    
    dwSize /= sizeof( DWORD ) ;
    
    // Loop through each ProcID.
    for( dwIndex = 0 ; dwIndex < dwSize ; dwIndex++ )
    {
        szFileName[0] = 0 ;
        hProcess = OpenProcess(
            PROCESS_QUERY_INFORMATION | PROCESS_VM_READ,
            FALSE, lpdwPIDs[ dwIndex ] ) ;
        if( hProcess != NULL )
        {
            if( m_lpfEnumProcessModules( hProcess, &hMod,
                sizeof( hMod ), &dwSize2 ) )
            {
                // Get Full pathname:
                if( !m_lpfGetModuleFileNameEx( hProcess, hMod,
                    szFileName, sizeof( szFileName ) ) )
                {
                    szFileName[0] = 0 ;
                }
            }
            CloseHandle( hProcess ) ;
        }
        DWORD lastErr = GetLastError();
        if(!lpProc( lpdwPIDs[dwIndex], 0, szFileName, lParam))
            break ;
        if( _stricmp( szFileName+(strlen(szFileName)-9),
            "NTVDM.EXE")==0)
        {
            sInfo.dwPID = lpdwPIDs[dwIndex] ;
            sInfo.lpProc = lpProc ;
            sInfo.lParam = lParam ;
            sInfo.bEnd = FALSE ;
            try
            {
                m_lpfVDMEnumTaskWOWEx( lpdwPIDs[dwIndex],
                    (TASKENUMPROCEX) Enum16,
                    (LPARAM) &sInfo);
            }
            catch(...)
            {
                continue;
            }
            if(sInfo.bEnd)
                break ;
        }
    }
    
    HeapFree( GetProcessHeap(), 0, lpdwPIDs ) ;
    
    return TRUE ;
}


BOOL CProcessEnumerator::Enum16( 
                DWORD dwThreadId, 
                WORD hMod16, 
                WORD hTask16,
                PSZ pszModName, 
                PSZ pszFileName, 
                LPARAM lpUserDefined )
{
  BOOL bRet ;

  ENUMINFOSTRUCT *psInfo = (ENUMINFOSTRUCT *)lpUserDefined ;

  bRet = psInfo->lpProc( psInfo->dwPID, 
         hTask16, pszFileName, psInfo->lParam ) ;

  if(!bRet)
  {
     psInfo->bEnd = TRUE ;
  }

  return !bRet;
} 
//

+ Good luck to all
