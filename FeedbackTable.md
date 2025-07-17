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
