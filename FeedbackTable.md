# FeedbackTable

Componente React para exibição, filtragem e paginação de tabelas customizadas utilizando [@tanstack/react-table](https://tanstack.com/table/latest). Permite ordenação, filtro por coluna, seleção de linhas, destaque de métricas e controle de visibilidade de colunas.

---

## Sumário

- [Descrição Geral](#descrição-geral)
- [Props](#props)
- [Funcionamento](#funcionamento)
- [Funcionalidades](#funcionalidades)
- [Destaque de Métricas](#destaque-de-métricas)
- [Paginação](#paginação)
- [Filtragem](#filtragem)
- [Dependências](#dependências)
- [Exemplo de Uso](#exemplo-de-uso)
- [Código Fonte](#código-fonte)
- [Customização](#customização)

---

## Descrição Geral

O `FeedbackTable` é um componente de tabela dinâmica que recebe dados genéricos e colunas definidas pelo usuário. Ele permite mostrar, filtrar, ordenar e paginar informações, além de destacar colunas de métricas específicas. Seu foco é ser flexível para qualquer tipo de dado e visualização.

---

## Props

```typescript
interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[]; // Definição das colunas da tabela (tanstack)
  data: TData[];                       // Array de dados a serem exibidos
}
```

---

## Funcionamento

- Utiliza `useReactTable` do Tanstack para gerenciar estado da tabela: ordenação, filtros, paginação, visibilidade e seleção de linhas.
- Renderiza cabeçalho e células dinamicamente com `flexRender`.
- Permite filtragem por campo específico (exemplo: 'guia'), controle de linhas por página e destaque de colunas de métricas.
- Mostra mensagem customizada quando não há resultados.

---

## Funcionalidades

- **Ordenação**: Clica no cabeçalho para ordenar colunas (default por 'total', decrescente).
- **Filtragem**: Campo input para filtrar por coluna específica ('guia').
- **Destaque de Métricas**: Botão para destacar colunas de métricas ('up', 'down', 'total', 'percentage').
- **Paginação**: Navegação por páginas e seleção de quantidade de linhas exibidas.
- **Seleção de Linhas**: Permite selecionar linhas.
- **Controle de Visibilidade**: Permite esconder/exibir colunas dinamicamente (via estado, customizável).
- **Feedback Visual**: Colunas destacadas recebem estilos visuais diferenciados.

---

## Destaque de Métricas

Ao ativar o botão "Destacar Métricas", as colunas `up`, `down`, `total` e `percentage` recebem estilos visuais para facilitar a análise.

---

## Paginação

- Botões para navegar entre páginas (primeira, anterior, próxima, última).
- Exibe página atual, total de páginas e resultados visíveis.
- Permite ajustar número de linhas por página (10, 20, 30, 40, 50).

---

## Filtragem

- Campo de filtro para buscar por valor da coluna `guia`.
- Mostra mensagem "Nenhum resultado encontrado" se não houver dados após filtragem.

---

## Dependências

- **React** (`useState`)
- **Tanstack React Table** (`@tanstack/react-table`)
- **Lucide React** (`Search`, `ChevronLeft`, `ChevronRight`, `ChevronsLeft`, `ChevronsRight`, `ListFilter`)
- **Componentes customizados**:
  - `Input` (campo de texto)
  - `Button` (botão)
  - `Table`, `TableBody`, `TableCell`, `TableHead`, `TableHeader`, `TableRow` (tabela)
  - `Select`, `SelectContent`, `SelectItem`, `SelectTrigger`, `SelectValue` (dropdown de seleção)

---

## Exemplo de Uso

```tsx
import FeedbackTable from '@/components/FeedbackTable';

const columns = [/* Definições conforme ColumnDef */];
const data = [/* Array de dados */];

<FeedbackTable columns={columns} data={data} />
```

---

## Código Fonte

```typescript
// ... (código completo do componente, clique para expandir)
```
<details>
<summary>Visualizar código</summary>

```typescript
'use client';

import {
	type ColumnDef,
	type ColumnFiltersState,
	type SortingState,
	type VisibilityState,
	flexRender,
	getCoreRowModel,
	getFilteredRowModel,
	getPaginationRowModel,
	getSortedRowModel,
	useReactTable,
} from '@tanstack/react-table';

import {
	Table,
	TableBody,
	TableCell,
	TableHead,
	TableHeader,
	TableRow,
} from '@/components/ui/table';

import { useState } from 'react';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Search, ChevronLeft, ChevronRight, ChevronsLeft, ChevronsRight, ListFilter } from 'lucide-react';
import {
	Select,
	SelectContent,
	SelectItem,
	SelectTrigger,
	SelectValue,
} from '@/components/ui/select';

interface DataTableProps<TData, TValue> {
	columns: ColumnDef<TData, TValue>[];
	data: TData[];
}

export default function FeedbackTable<TData, TValue>({
	columns,
	data,
}: DataTableProps<TData, TValue>) {
	const [sorting, setSorting] = useState<SortingState>([
		{ id: 'total', desc: true }
	]);
	const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
	const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({});
	const [rowSelection, setRowSelection] = useState({});
	const [highlightColumns, setHighlightColumns] = useState(false);

	const table = useReactTable({
		data,
		columns,
		onSortingChange: setSorting,
		onColumnFiltersChange: setColumnFilters,
		getCoreRowModel: getCoreRowModel(),
		getPaginationRowModel: getPaginationRowModel(),
		getSortedRowModel: getSortedRowModel(),
		getFilteredRowModel: getFilteredRowModel(),
		onColumnVisibilityChange: setColumnVisibility,
		onRowSelectionChange: setRowSelection,
		state: {
			sorting,
			columnFilters,
			columnVisibility,
			rowSelection,
		},
		initialState: {
			pagination: {
				pageSize: 10,
			},
		},
	});


	const shouldHighlightColumn = (columnId: string) => {
		return highlightColumns && ['up', 'down', 'total', 'percentage'].includes(columnId);
	};

	return (
		<div className="w-full p-6">
			<div className="flex items-center justify-between py-4">
				<div className="flex items-center gap-4">
					<div className="relative flex-1 max-w-md">
						<Search className="absolute left-2 top-2.5 h-4 w-4 text-muted-foreground" />
						<Input
							placeholder="Filtrar por documentação"
							value={(table.getColumn('guia')?.getFilterValue() as string) ?? ''}
							onChange={(event) =>
								table.getColumn('guia')?.setFilterValue(event.target.value)
							}
							className="pl-8"
						/>
					</div>
					
					<Button
						variant={highlightColumns ? "default" : "outline"}
						size="sm"
						onClick={() => setHighlightColumns(!highlightColumns)}
						className={`flex items-center gap-2 rounded-full ${
							highlightColumns 
								? 'bg-blue-600 text-white hover:bg-blue-700' 
								: 'border-blue-500 text-blue-700 hover:bg-blue-50 hover:text-blue-700'
						}`}
					>
						<ListFilter className="h-4 w-4" />
						{highlightColumns ? 'Remover Destaque' : 'Destacar Métricas'}
					</Button>
				</div>
				
				<div className="flex items-center gap-2">
					<span className="text-sm text-muted-foreground">Linhas por página:</span>
					<Select
						value={`${table.getState().pagination.pageSize}`}
						onValueChange={(value) => {
							table.setPageSize(Number(value));
						}}
					>
						<SelectTrigger className="h-8 w-[70px]">
							<SelectValue placeholder={table.getState().pagination.pageSize} />
						</SelectTrigger>
						<SelectContent side="top">
							{[10, 20, 30, 40, 50].map((pageSize) => (
								<SelectItem key={pageSize} value={`${pageSize}`}>
									{pageSize}
								</SelectItem>
							))}
						</SelectContent>
					</Select>
				</div>
			</div>

			<div className="rounded-md border">
				<Table>
					<TableHeader>
						{table.getHeaderGroups().map((headerGroup) => (
							<TableRow key={headerGroup.id} className="bg-gray-50">
								{headerGroup.headers.map((header) => {
									const isHighlighted = shouldHighlightColumn(header.column.id);
									return (
										<TableHead 
											key={header.id} 
											className={`font-semibold transition-colors duration-200 ${
												isHighlighted 
													? 'bg-blue-100 border-blue-200 text-blue-800' 
													: ''
											}`}
										>
											{header.isPlaceholder
												? null
												: flexRender(
														header.column.columnDef.header,
														header.getContext()
												)}
										</TableHead>
									);
								})}
							</TableRow>
						))}
					</TableHeader>
					<TableBody>
						{table.getRowModel().rows?.length ? (
							table.getRowModel().rows.map((row) => (
								<TableRow
									key={row.id}
									data-state={row.getIsSelected() && 'selected'}
									className="hover:bg-gray-50"
								>
									{row.getVisibleCells().map((cell) => {
										const isHighlighted = shouldHighlightColumn(cell.column.id);
										return (
											<TableCell 
												key={cell.id}
												className={`transition-colors duration-200 ${
													isHighlighted 
														? 'bg-blue-50 border-blue-100' 
														: ''
												}`}
											>
												{flexRender(
													cell.column.columnDef.cell,
													cell.getContext()
												)}
											</TableCell>
										);
									})}
								</TableRow>
							))
						) : (
							<TableRow>
								<TableCell
									colSpan={columns.length}
									className="h-24 text-center"
								>
									<div className="flex flex-col items-center gap-2">
										<Search className="h-8 w-8 text-gray-400" />
										<span className="text-gray-500">Nenhum resultado encontrado.</span>
									</div>
								</TableCell>
							</TableRow>
						)}
					</TableBody>
				</Table>
			</div>

			<div className="flex items-center justify-between space-x-2 py-4">
				<div className="flex-1 text-sm text-muted-foreground">
					Mostrando {table.getState().pagination.pageIndex * table.getState().pagination.pageSize + 1} a{' '}
					{Math.min(
						(table.getState().pagination.pageIndex + 1) * table.getState().pagination.pageSize,
						table.getFilteredRowModel().rows.length
					)}{' '}
					de {table.getFilteredRowModel().rows.length} resultado(s)
				</div>
				
				<div className="flex items-center space-x-2">
					<Button
						variant="outline"
						className="hidden h-8 w-8 p-0 lg:flex"
						onClick={() => table.setPageIndex(0)}
						disabled={!table.getCanPreviousPage()}
					>
						<span className="sr-only">Ir para primeira página</span>
						<ChevronsLeft className="h-4 w-4" />
					</Button>
					<Button
						variant="outline"
						className="h-8 w-8 p-0"
						onClick={() => table.previousPage()}
						disabled={!table.getCanPreviousPage()}
					>
						<span className="sr-only">Ir para página anterior</span>
						<ChevronLeft className="h-4 w-4" />
					</Button>
					<div className="flex w-[100px] items-center justify-center text-sm font-medium">
						Página {table.getState().pagination.pageIndex + 1} de{' '}
						{table.getPageCount()}
					</div>
					<Button
						variant="outline"
						className="h-8 w-8 p-0"
						onClick={() => table.nextPage()}
						disabled={!table.getCanNextPage()}
					>
						<span className="sr-only">Ir para próxima página</span>
						<ChevronRight className="h-4 w-4" />
					</Button>
					<Button
						variant="outline"
						className="hidden h-8 w-8 p-0 lg:flex"
						onClick={() => table.setPageIndex(table.getPageCount() - 1)}
						disabled={!table.getCanNextPage()}
					>
						<span className="sr-only">Ir para última página</span>
						<ChevronsRight className="h-4 w-4" />
					</Button>
				</div>
			</div>
		</div>
	);
}
```
</details>

---

## Customização

- Defina colunas e dados conforme necessidade via props.
- Edite estilos e campos de filtro conforme sua aplicação.
- Para destacar outras métricas, altere a função `shouldHighlightColumn`.
- Integre com backend ou outras fontes de dados conforme necessário.

---

## Observações

- Ideal para dashboards, painéis administrativos ou qualquer visualização tabular customizada.
- Fácil de expandir: adicione novas funcionalidades alterando os estados ou integrando novos recursos do Tanstack Table.

---

## Autor

Documentação gerada por [Phillipe-Hugo](https://github.com/Phillipe-Hugo).
