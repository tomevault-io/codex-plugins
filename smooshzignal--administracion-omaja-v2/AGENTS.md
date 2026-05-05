# Copilot Instructions

## Project Guidelines
- En este proyecto, los filtros de sucursal y destino deben comparar exactamente los textos proporcionados en los ComboBox, igual que en Form1; no usar mapeos ni normalización adicional salvo la lógica ya existente allí.
- Todos los métodos nuevos relacionados con la funcionalidad de Contraloria deben incluir explícitamente la palabra 'Contraloria' en su nombre para mantenerlos aislados de la lógica existente de Form1 y no afectar código actual.
- El flujo de importación de guías debe conservar la lógica actual de insertar si el folio no existe y actualizar si ya existe; no reemplazarlo por un cambio de comportamiento funcional.
- En Contraloria, los filtros correctos son: 
  - checkBox1=TipoCobro 'Pagado'; 
  - checkBox2=TipoCobro 'Por Cobrar'; 
  - checkBox3=TipoCobro 'Cancelado' o EstatusGuia 'Cancelado'; 
  - checkBox4=EstatusGuia 'Completado' (buscando solo en la columna EstatusGuia con el valor 'Completado'); 
  - checkBox5=EstatusGuia 'Entregada' (buscando solo en la columna EstatusGuia con el valor 'Entregada'); 
  - checkBox6=Factura 'Facturada' (filtrar guías cuya columna Factura contiene un dato); 
  - checkBox7=Factura 'No facturada' (filtrar guías cuya columna Factura está vacía o nula). Estos filtros deben ser ejecutados desde buttonFiltrarTipodeCobro y usar consulta SQL exacta sobre la tabla Guias.
- En Contraloria no se debe destruir la funcionalidad existente de filtros con checkboxes y DateTimePicker; los cambios nuevos deben agregarse sin afectar esos filtros.

---
> Source: [smooshzignal/Administracion-OMAJA-V2](https://github.com/smooshzignal/Administracion-OMAJA-V2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
