<script lang="ts">
	import { dndzone } from 'svelte-dnd-action';
	import { flip } from 'svelte/animate';
	import { PDFDocument } from 'pdf-lib';
	import { UploadCloud, FileDown, Loader2, GripVertical, Trash2 } from 'lucide-svelte';
	
	interface ImageItem {
		id: string;
		file: File;
		previewUrl: string;
	}

	let items = $state<ImageItem[]>([]);
	let isConverting = $state(false);
	let pdfUrl = $state<string | null>(null);
	let selectedImage = $state<string | null>(null);
	let fileInput: HTMLInputElement;
	
	const flipDurationMs = 200;
	
	function handleFilesSelect(event: Event | { dataTransfer: DataTransfer, target: any }) {
		const target = event.target as HTMLInputElement;
		const dataTransfer = (event as any).dataTransfer;
		const files = Array.from(target?.files || dataTransfer?.files || []);
		
		const newItems = files
			.filter(f => f.type.startsWith('image/'))
			.map(file => ({
				id: crypto.randomUUID(),
				file,
				previewUrl: URL.createObjectURL(file)
			}));
			
		items = [...items, ...newItems];
		
		if (fileInput) fileInput.value = '';
	}

	function handleDropAreaDrop(e: DragEvent) {
		e.preventDefault();
		handleFilesSelect(e);
	}

	function handleDndConsider(e: CustomEvent<{items: ImageItem[]}>) {
		items = e.detail.items;
	}
	
	function handleDndFinalize(e: CustomEvent<{items: ImageItem[]}>) {
		items = e.detail.items;
	}

	function removeItem(id: string) {
		items = items.filter(i => i.id !== id);
	}
	
	function openImage(url: string) {
		selectedImage = url;
	}
	
	function closeImage() {
		selectedImage = null;
	}
	
	async function generatePDF() {
		if (items.length === 0) return;
		
		isConverting = true;
		pdfUrl = null;
		
		try {
			const pdfDoc = await PDFDocument.create();
			
			for (const item of items) {
				const imageBytes = await item.file.arrayBuffer();
				let image;
				
				if (item.file.type === 'image/jpeg' || item.file.type === 'image/jpg') {
					image = await pdfDoc.embedJpg(imageBytes);
				} else if (item.file.type === 'image/png') {
					image = await pdfDoc.embedPng(imageBytes);
				} else {
					continue; // Unsupported format skip
				}
				
				const page = pdfDoc.addPage();
				const { width: pageWidth, height: pageHeight } = page.getSize();
				
				// Calculate scaling factor to fit the image on the page
				const imgDims = image.scaleToFit(pageWidth, pageHeight);
				
				// Center the image on the page
				page.drawImage(image, {
					x: pageWidth / 2 - imgDims.width / 2,
					y: pageHeight / 2 - imgDims.height / 2,
					width: imgDims.width,
					height: imgDims.height,
				});
			}
			
			const pdfBytes = await pdfDoc.save();
			const blob = new Blob([pdfBytes], { type: 'application/pdf' });
			pdfUrl = URL.createObjectURL(blob);
		} catch (error) {
			console.error("Error generating PDF:", error);
			alert("PDF 생성 중 오류가 발생했습니다.");
		} finally {
			isConverting = false;
		}
	}
</script>

<div class="min-h-screen bg-neutral-950 text-neutral-50 p-4 md:p-8 font-sans selection:bg-blue-500/30">
	<div class="max-w-4xl mx-auto">
		<header class="text-center mb-10 pt-8">
			<h1 class="text-4xl md:text-6xl font-extrabold tracking-tight mb-4 bg-gradient-to-r from-blue-400 via-indigo-400 to-emerald-400 bg-clip-text text-transparent">
				Image to PDF
			</h1>
			<p class="text-neutral-400 text-lg">이미지를 드래그하여 순서를 맞추고 PDF로 변환하세요. <br class="hidden md:block"/> 모든 작업은 브라우저에서 안전하게 처리됩니다.</p>
		</header>
		
		<!-- Dropzone -->
		<!-- svelte-ignore a11y_click_events_have_key_events -->
		<!-- svelte-ignore a11y_no_static_element_interactions -->
		<div 
			class="border-2 border-dashed border-neutral-700 hover:border-blue-500 rounded-3xl p-12 text-center transition-all bg-neutral-900/50 hover:bg-neutral-800/50 backdrop-blur-sm cursor-pointer mb-12 shadow-xl"
			on:dragover|preventDefault
			on:drop={handleDropAreaDrop}
			on:click={() => fileInput.click()}
		>
			<input 
				type="file" 
				multiple 
				accept="image/png, image/jpeg, image/jpg" 
				class="hidden" 
				on:change={handleFilesSelect}
				bind:this={fileInput}
			/>
			<div class="bg-blue-500/10 w-24 h-24 rounded-full flex items-center justify-center mx-auto mb-6">
				<UploadCloud class="w-12 h-12 text-blue-400" />
			</div>
			<p class="text-2xl font-bold mb-3 text-neutral-200">클릭하거나 이미지를 여기로 드래그하세요</p>
			<p class="text-neutral-500 text-base">PNG, JPG 포맷 지원</p>
		</div>

		{#if items.length > 0}
			<div class="mb-12 bg-neutral-900/40 p-6 md:p-8 rounded-3xl border border-neutral-800 shadow-2xl">
				<div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 mb-6">
					<h2 class="text-2xl font-bold flex items-center gap-2">
						선택된 이미지 
						<span class="bg-blue-500/20 text-blue-400 text-sm px-3 py-1 rounded-full">{items.length}</span>
					</h2>
					<button class="text-sm text-red-400 hover:text-red-300 transition-colors font-medium hover:bg-red-400/10 px-4 py-2 rounded-lg" on:click={() => items = []}>전체 삭제</button>
				</div>
				
				<section 
					use:dndzone={{items, flipDurationMs, dropTargetStyle: {outline: '2px solid #3b82f6', borderRadius: '0.75rem'}}} 
					on:consider={handleDndConsider} 
					on:finalize={handleDndFinalize}
					class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4 min-h-[120px]"
				>
					{#each items as item (item.id)}
						<div class="relative group bg-neutral-800 rounded-xl overflow-hidden aspect-[3/4] border border-neutral-700 shadow-lg" animate:flip={{duration: flipDurationMs}}>
							<img src={item.previewUrl} alt="preview" class="w-full h-full object-cover" />
							<!-- svelte-ignore a11y_click_events_have_key_events -->
							<!-- svelte-ignore a11y_no_static_element_interactions -->
							<div 
								class="absolute inset-0 bg-gradient-to-t from-black/80 via-black/20 to-black/40 opacity-0 group-hover:opacity-100 transition-all duration-300 flex flex-col justify-between p-3 cursor-pointer"
								onclick={() => openImage(item.previewUrl)}
							>
								<div class="flex justify-end">
									<button 
										class="p-2 bg-red-500/80 hover:bg-red-500 rounded-full text-white transition-colors transform hover:scale-110 active:scale-95 shadow-lg"
										onclick={(e) => { e.stopPropagation(); removeItem(item.id); }}
										aria-label="삭제"
									>
										<Trash2 class="w-4 h-4" />
									</button>
								</div>
								<div class="flex justify-center flex-1 items-center">
									<GripVertical class="w-10 h-10 text-white/90 drop-shadow-md cursor-grab active:cursor-grabbing" />
								</div>
								<div class="text-xs text-white/90 font-medium truncate drop-shadow-md">
									{item.file.name}
								</div>
							</div>
						</div>
					{/each}
				</section>
			</div>
			
			<div class="flex flex-col items-center gap-4">
				{#if isConverting}
					<button disabled class="flex items-center gap-3 bg-neutral-800 text-neutral-400 px-10 py-5 rounded-2xl font-bold text-xl w-full md:w-auto justify-center cursor-not-allowed border border-neutral-700">
						<Loader2 class="w-6 h-6 animate-spin text-blue-500" />
						PDF 변환 중...
					</button>
				{:else if pdfUrl}
					<a 
						href={pdfUrl} 
						download="merged-document.pdf" 
						class="flex items-center gap-3 bg-gradient-to-r from-emerald-500 to-emerald-400 hover:from-emerald-400 hover:to-emerald-300 text-white px-10 py-5 rounded-2xl font-bold text-xl w-full md:w-auto justify-center shadow-xl shadow-emerald-500/20 transition-all transform hover:-translate-y-1 hover:shadow-emerald-500/30 active:scale-95"
					>
						<FileDown class="w-6 h-6" />
						PDF 다운로드
					</a>
					<button 
						on:click={() => pdfUrl = null} 
						class="text-neutral-400 hover:text-white mt-2 underline underline-offset-4 text-sm transition-colors"
					>
						새로운 파일로 다시 만들기
					</button>
				{:else}
					<button 
						on:click={generatePDF} 
						class="flex items-center gap-3 bg-gradient-to-r from-blue-600 to-blue-500 hover:from-blue-500 hover:to-blue-400 text-white px-10 py-5 rounded-2xl font-bold text-xl w-full md:w-auto justify-center shadow-xl shadow-blue-500/20 transition-all transform hover:-translate-y-1 hover:shadow-blue-500/30 active:scale-95"
					>
						PDF 만들기
					</button>
				{/if}
			</div>
		{/if}
	</div>
</div>

{#if selectedImage}
	<!-- svelte-ignore a11y_click_events_have_key_events -->
	<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
	<div 
		class="fixed inset-0 z-50 flex items-center justify-center bg-black/90 p-4 md:p-12 backdrop-blur-sm transition-opacity"
		onclick={closeImage}
		role="dialog"
	>
		<img 
			src={selectedImage} 
			alt="Enlarged preview" 
			class="max-w-full max-h-full object-contain rounded-xl shadow-2xl" 
			onclick={(e) => e.stopPropagation()} 
		/>
		<button 
			class="absolute top-4 right-4 md:top-8 md:right-8 p-3 text-white/70 hover:text-white bg-black/50 hover:bg-black/80 rounded-full transition-all transform hover:scale-110"
			onclick={closeImage}
			aria-label="닫기"
		>
			<svg xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>
		</button>
	</div>
{/if}
