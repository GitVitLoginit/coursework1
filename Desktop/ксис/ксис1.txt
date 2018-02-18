#include "stdafx.h"
#include <stdio.h>
#include <winsock2.h>
#include <iphlpapi.h>
#include <winnetwk.h>
#include <iostream>
#pragma comment(lib, "iphlpapi.lib")
#pragma comment(lib, "Mpr.lib")
using namespace std;
// выводит 6 байт, начиная с addr в консоль
void PrintMACaddress(BYTE *addr) {
for (int i = 0; i < 6; ++i) {
printf("%02x%c", *addr++, (i < 5) ? '-' : '\n');
}
}

// печатаем МАК адреса всех адаптеров
static void GetMACaddress() {
IP_ADAPTER_INFO AdapterInfo[16]; // информация о 16 адаптерах макс
DWORD dwBufLen = sizeof(AdapterInfo); // кол-во байт в буффере

DWORD dwStatus = GetAdaptersInfo(AdapterInfo, &dwBufLen);
if (dwStatus != ERROR_SUCCESS) {
printf("GetAdaptersInfo failed. err=%d\n", GetLastError());
return;
}

PIP_ADAPTER_INFO pAdapterInfo = AdapterInfo; // информация о текущем адаптере
while (pAdapterInfo) { // пока есть информация об адаптерах
PrintMACaddress(pAdapterInfo->Address); // печатаем MAC
pAdapterInfo = pAdapterInfo->Next; // переходим к информации о следующем адаптере
}
}

BOOL GetListNet(LPNETRESOURCE* lpNet1)
{
LPNETRESOURCE lpNet2;
HANDLE hEnum;
DWORD buff = 16384;
DWORD cnt = -1;
DWORD dwResult;

dwResult = WNetOpenEnum(RESOURCE_GLOBALNET, RESOURCETYPE_ANY, 0, *lpNet1, &hEnum);

if (dwResult == NO_ERROR)
{
lpNet2 = (LPNETRESOURCE)GlobalAlloc(GPTR, buff);
if (lpNet2 == NULL)
return FALSE;

do
{
ZeroMemory(lpNet2, buff);

dwResult = WNetEnumResource(hEnum, &cnt, lpNet2, &buff);

if (dwResult == NO_ERROR)
{

for (DWORD i = 0; i < cnt; i++)
{
printf("NETRESOURCE[%d] Scope: ", i);

switch (lpNet2->dwScope) {
case (RESOURCE_CONNECTED): //все подключённые ресурсы
printf("connected\n");
break;
case (RESOURCE_GLOBALNET):
printf("all resources\n");
break;
case (RESOURCE_REMEMBERED):
printf("remembered\n");
break;
default:
printf("unknown scope %d\n", lpNet2->dwScope);
break;
}

printf("NETRESOURCE[%d] Type: ", i);
switch (lpNet2->dwType) {
case (RESOURCETYPE_ANY):
printf("any\n");
break;
case (RESOURCETYPE_DISK):
printf("disk\n");
break;
case (RESOURCETYPE_PRINT):
printf("print\n");
break;
default:
printf("unknown type %d\n", lpNet2->dwType);
break;
}

printf("NETRESOURCE[%d] DisplayType: ", i);
switch (lpNet2->dwDisplayType) {
case (RESOURCEDISPLAYTYPE_GENERIC):
printf("generic\n");
break;
case (RESOURCEDISPLAYTYPE_DOMAIN):
printf("domain\n");
break;
case (RESOURCEDISPLAYTYPE_SERVER):
printf("server\n");
break;
case (RESOURCEDISPLAYTYPE_SHARE):
printf("share\n");
break;
case
(RESOURCEDISPLAYTYPE_FILE):
printf("file\n");
break;
case (RESOURCEDISPLAYTYPE_GROUP):
printf("group\n");
break;
case (RESOURCEDISPLAYTYPE_NETWORK):
printf("network\n");
break;
default:
printf("unknown display type %d\n", lpNet2->dwDisplayType);
break;
}

printf("NETRESOURCE[%d] Usage: 0x%x = ", i, lpNet2->dwUsage);
if (lpNet2->dwUsage & RESOURCEUSAGE_CONNECTABLE) //все подключённые рес-сы
printf("connectable ");
if (lpNet2->dwUsage & RESOURCEUSAGE_CONTAINER) //рес-сы контейнеры
printf("container ");
printf("\n");

printf("NETRESOURCE[%d] Localname: %S\n", i, lpNet2->lpLocalName);
printf("NETRESOURCE[%d] Remotename: %S\n", i, lpNet2->lpRemoteName);
printf("NETRESOURCE[%d] Comment: %S\n", i, lpNet2->lpComment);
printf("NETRESOURCE[%d] Provider: %S\n", i, lpNet2->lpProvider);
printf("\n");
if (RESOURCEUSAGE_CONTAINER == (lpNet2->dwUsage & RESOURCEUSAGE_CONTAINER))
{
if (!GetListNet(&lpNet2))
{

}
}
}

}
else return FALSE;
} while (dwResult != ERROR_NO_MORE_ITEMS);

if (WNetCloseEnum(hEnum) != NO_ERROR)
return FALSE;

}
else return FALSE;

return TRUE;
}

void main() {
char* Buffer = new char;
GetMACaddress();

LPNETRESOURCE
lpRun;
DWORD buff = 16384;

lpRun = (LPNETRESOURCE)GlobalAlloc(GPTR, buff);

GetListNet(&lpRun);

getchar();
getchar();
} 
