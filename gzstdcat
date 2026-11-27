from argparse import ArgumentParser
import sys
from zstandard import ZstdDecompressor, ZstdError

MAGIC = b'\x28\xB5\x2F\xFD'
BUFFER = 1024**2

def main():
    parser = ArgumentParser()
    parser.add_argument('input', nargs='*', default=['-'])
    parser.add_argument('-v', action='store_true')
    args = parser.parse_args()
    out = sys.stdout.buffer
    buf = bytearray(BUFFER)
    mv = memoryview(buf)
    dctx = ZstdDecompressor()
    for path in args.input:
        inp = sys.stdin.buffer if path == '-' else open(path, 'rb')
        dc = None
        frame = None
        consumed = 0
        leftover = b''
        while size := inp.readinto(mv[len(leftover):]):
            size += len(leftover)
            leftover = b''
            for n in range(len(MAGIC)-1, 0, -1):
                if buf[size-n:size] == MAGIC[:n]:
                    leftover = buf[size-n:size]
                    size -= n
                    break
            pos = i = 0
            while True:
                i = buf.find(MAGIC, i, size)
                if dc:
                    try:
                        end = size if i == -1 else i
                        out.write(dc.decompress(mv[pos:end]))
                        if dc.eof:
                            dc = None
                    except ZstdError as e:
                        dc = None
                        if args.v:
                            print(f'{path}: broken frame at offset {frame}: {e}', file=sys.stderr)
                if i == -1:
                    break
                if not dc:
                    dc = dctx.decompressobj(read_across_frames=False)
                    frame = consumed + i
                pos = i
                i += 1
            consumed += size
            if leftover:
                buf[:len(leftover)] = leftover
        if dc:
            try:
                out.write(dc.decompress(leftover))
                if not dc.eof and args.v:
                    print(f'{path}: truncated frame at offset {frame}', file=sys.stderr)
            except ZstdError as e:
                if args.v:
                    print(f'{path}: broken frame at offset {frame}: {e}', file=sys.stderr)
        inp.close()

if __name__ == '__main__':
    main()
