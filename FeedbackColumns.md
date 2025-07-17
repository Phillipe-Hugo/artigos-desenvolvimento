'use client';
import { type ColumnDef } from '@tanstack/react-table';
import { ArrowUpIcon, ArrowDownIcon, MessageSquare } from 'lucide-react';
import { Button } from '@/components/ui/button';


type FeedbackData = {
	id: string;
	up: number;
	down: number;
	documentation: {
		id: string;
		created_by?: string | null;
		updated_by?: string | null;
		created_at?: Date | null;
		updated_at?: Date | null;
		viewable_by?: unknown;
		editable_by?: unknown;
		deletable_by?: unknown;
		translations: {
			title: string;
			id: string;
			language: string;
			content: string;
			documentation_id: string;
			translationId: string;
		}[];
	} | null;
	comments?: {
		id: number;
		documentationId: string;
		comment: string;
		userId: string;
		user?: {
			id: string;
			name: string | null;
			email: string;
		};
	}[];
};

export const feedbackColumns: ColumnDef<FeedbackData>[] = [
	{
		id: 'guia',
		accessorFn: (row) => {
			const ptTranslation = row.documentation?.translations?.find(
				t => t.language === 'pt' && t.title && t.title.trim() !== ''
			);
			const anyTranslation = row.documentation?.translations?.find(
				t => t.title && t.title.trim() !== ''
			);
			return ptTranslation?.title ?? anyTranslation?.title ?? 'Documentação sem título';
		},
		header: ({ column }) => (
			<Button
				variant="ghost"
				onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
				className="font-semibold text-left justify-start p-0 h-auto"
			>
				GUIA
			</Button>
		),
		cell: ({ row }) => {
			const ptTranslation = row.original.documentation?.translations?.find(
				t => t.language === 'pt' && t.title && t.title.trim() !== ''
			);
			const anyTranslation = row.original.documentation?.translations?.find(
				t => t.title && t.title.trim() !== ''
			);
			const title = ptTranslation?.title ?? anyTranslation?.title ?? 'Documentação sem título';
			
			return (
				<div className="flex items-center">
					<div>
						<p className="font-medium text-gray-900">{title}</p>
						<p className="text-sm text-gray-500">ID: {row.original.id}</p>
					</div>
				</div>
			);
		},
		filterFn: (row, columnId, filterValue) => {
			const ptTranslation = row.original.documentation?.translations?.find(
				t => t.language === 'pt' && t.title && t.title.trim() !== ''
			);
			const anyTranslation = row.original.documentation?.translations?.find(
				t => t.title && t.title.trim() !== ''
			);
			const title = ptTranslation?.title ?? anyTranslation?.title ?? '';
			return title.toLowerCase().includes((filterValue as string).toLowerCase());
		},
	},
	{
		id: 'up',
		accessorKey: 'up',
		header: ({ column }) => (
			<Button
				variant="ghost"
				onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
				className="font-semibold text-left justify-start p-0 h-auto"
			>
				UP
			</Button>
		),
		cell: ({ row }) => {
			const upCount = row.original.up || 0;
			return (
				<div className="flex items-center gap-2">
					<ArrowUpIcon className="h-4 w-4 text-green-600" />
					<span className="font-semibold text-green-600">{upCount}</span>
				</div>
			);
		},
	},
	{
		id: 'down',
		accessorKey: 'down',
		header: ({ column }) => (
			<Button
				variant="ghost"
				onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
				className="font-semibold text-left justify-start p-0 h-auto"
			>
				DOWN
			</Button>
		),
		cell: ({ row }) => {
			const downCount = row.original.down || 0;
			return (
				<div className="flex items-center gap-2">
					<ArrowDownIcon className="h-4 w-4 text-red-600" />
					<span className="font-semibold text-red-600">{downCount}</span>
				</div>
			);
		},
	},
	{
		id: 'total',
		accessorFn: (row) => (row.up || 0) + (row.down || 0),
		header: ({ column }) => (
			<Button
				variant="ghost"
				onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
				className="font-semibold text-left justify-start p-0 h-auto"
			>
				TOTAL
			</Button>
		),
		cell: ({ row }) => {
			const total = (row.original.up || 0) + (row.original.down || 0);
			return (
				<div className="flex items-center">
					<span className="font-medium text-gray-900">{total}</span>
				</div>
			);
		},
	},
	{
		id: 'percentage',
		accessorFn: (row) => {
			const total = (row.up || 0) + (row.down || 0);
			if (total === 0) return 0;
			return ((row.up || 0) / total) * 100;
		},
		header: ({ column }) => (
			<Button
				variant="ghost"
				onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
				className="font-semibold text-left justify-start p-0 h-auto"
			>
				APROVAÇÃO
			</Button>
		),
		cell: ({ row }) => {
			const total = (row.original.up || 0) + (row.original.down || 0);
			if (total === 0) {
				return <span className="text-gray-400">-</span>;
			}
			const percentage = ((row.original.up || 0) / total) * 100;
			const colorClass = percentage >= 70 
				? 'text-green-600' 
				: percentage >= 50 
					? 'text-yellow-600' 
					: 'text-red-600';
			return (
				<span className={`font-medium ${colorClass}`}>
					{percentage.toFixed(1)}%
				</span>
			);
		},
	},
